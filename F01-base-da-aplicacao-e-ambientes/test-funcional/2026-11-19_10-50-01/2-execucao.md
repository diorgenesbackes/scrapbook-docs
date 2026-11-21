# Resultado da validação funcional da F01-STORY-001

**Reprovada no laboratório local.** O PostgreSQL recebeu as 24 tabelas previstas e a identidade da aplicação obteve a matriz DML esperada. A instalação e a reversão aceitaram um schema com um tipo enumerado alheio. A pré-validação também aceitou uma concessão administrativa sobre uma coluna de sistema do controle do Liquibase. [Plano de validação](1-plano.md) · [Cartão](https://trello.com/c/gSGvlzEZ) · [Spec §6–7](../../spec.md#6-modelo-de-dados)

## Ambiente e execução

Em 19/11/2026, a execução usou PostgreSQL 18.6 (`server_version_num` `180006`), Liquibase 4.33.0 e Java 26.0.1. O runner usou a imagem PostgreSQL fixada por digest em `config/versions.json`, contêineres próprios com portas dinâmicas em loopback e dados sintéticos. O código foi extraído do merge `1adeced61e418c3150d3e47d139091835250906c` do Liquibase, com a mesma árvore do commit `107b5cb` da história. O changeset validado foi `f01-story-001-base`, autor `scrapbook`, arquivo `changelogs/sprint-1/changelog.yaml`. A revisão documental foi `b45b3197cd6fc8552699317fe229b073e50b6cb6`.

| Execução | Resultado |
| --- | --- |
| `python3 tests/test_baseline.py` | Código 0. 18 testes aprovados em 112,071 s, sem skips. |
| Verificações temporárias `python3 tests/acceptance_f01.py`, com o mesmo runner | Código 1. Três verificações passaram e três falharam em 40,932 s, sem skips. |

A primeira rodada complementar já reproduziu as duas falhas de catálogo com o tipo enumerado. Ela também revelou problemas na coleta de evidência. A leitura do dicionário capturava linhas de índices como colunas, e o teste de permissão mudava o ACL do schema depois de gravada a assinatura inicial. A rodada final corrigiu esses dois pontos no roteiro temporário e repetiu os seis casos sobre a mesma revisão do produto.

## Matriz de aceite

| Cenário | Resultado observado e evidência | Estado |
| --- | --- | --- |
| CA1.1 · Instalação | `validate` passou. `update-sql` apresentou o DDL sem criar tabelas de aplicação. O `update` criou as 24 tabelas exatas e os dois controles. O catálogo mostrou `scrapbook_owner` como proprietária sem login e o histórico registrou o changeset inicial. `test_install_single_schema_inventory`, `test_ca1_catalog`. | Aprovado |
| CA1.2 · Catálogo | As 415 colunas conferiram em nome, tipo e nulabilidade. A consulta verificou os defaults dos metadados e do lease, as 24 PKs identity, 34 FKs validadas com `RESTRICT`, 84 CHECKs validados e 109 índices válidos. A Outbox e o controle de mensagens recusaram duplicatas nos escopos previstos. `test_ca1_catalog`, `test_shared_technical_uniqueness`. | Aprovado |
| CA2.1 · Reaplicação | O segundo `update` preservou a assinatura e o histórico da base. `status` informou que a base estava atualizada. A evolução sintética pôde ser aplicada, reaplicada e revertida, retomando a assinatura da geração anterior. `test_reapply_preserves_catalog_and_detects_drift`, `test_schema_generation_can_be_reapplied_and_reverted`. | Aprovado |
| CA2.2 · Estado incompatível | Tabela incompatível, coluna extra, sequência alterada, auditoria `UNLOGGED` e histórico de outro changelog foram recusados. Um enum alheio preexistente foi aceito. O `update` retornou 0, criou as 24 tabelas e registrou o changeset. `test_ca2_preexisting_enum` e testes de drift da suíte. | Reprovado |
| CA2.3 · Falha de changeset | O Liquibase diagnosticou a relação ausente na mudança sintética. O DDL intermediário foi revertido, o histórico anterior foi preservado e o `update` da base voltou a passar. O rollback sintético incompleto também foi recusado e preservou a geração ativa. `test_failed_changeset_preserves_prior_history`, `test_incomplete_rollback_preserves_active_generation_and_history`. | Aprovado |
| CA3.1 · Matriz da aplicação | A consulta de privilégios efetivos conferiu as 26 tabelas. A aplicação gerou ID, inseriu e atualizou uma conta. CREATE, ALTER, DROP, leitura dos controles e assunção da proprietária foram recusados. O catálogo confirmou ausência de opção de concessão e de leitura de `ctid` para a credencial normal. `test_application_permissions_and_audit`, `test_ca3_matrix`. | Aprovado |
| CA3.2 · Auditoria | O INSERT de auditoria passou com a identidade da aplicação. SELECT, UPDATE, DELETE e TRUNCATE foram recusados. A role de consulta teve SELECT efetivo e privilégios de escrita ausentes. `test_application_permissions_and_audit`, `test_ca3_matrix`. | Aprovado |
| CA3.3 · Grant fora da matriz | Grants de tabela e coluna comum fora da matriz foram recusados. Com a concessão administrativa de SELECT apenas sobre `ctid`, o PostgreSQL reconheceu o privilégio da role sintética e o `update` retornou 0 com histórico intacto. `test_reapply_rejects_control_grants_outside_matrix`, `test_ca3_system_column_grant`. | Reprovado |
| CA4.1 · Reversão vazia | O rollback removeu as 24 tabelas, preservou o schema e os dois controles e esvaziou o histórico. Repetir o rollback retornou 0 e preservou esse estado. O `update` seguinte restaurou as 24 tabelas, a assinatura e o registro inicial. `test_rollback_and_reapply_empty_baseline`, `test_ca4_rollback_protections`. | Aprovado |
| CA4.2 · Proteções de reversão | O rollback recusou dados, tabela alheia e view dependente em outro schema, preservando a linha sintética, a tabela alheia, a view e o histórico nos respectivos casos. Com um enum alheio, retornou 0, removeu as 24 tabelas vazias e esvaziou o histórico. O enum permaneceu utilizável. `test_ca4_rollback_protections`, `test_ca4_enum_after_install`. | Reprovado |

## Divergências reproduzidas

### P2 · Tipo enumerado alheio passa pelas verificações de catálogo

Em um banco descartável já provisionado pelo bootstrap, criar `CREATE TYPE scrapbook.fixture_unmanaged_enum AS ENUM ('synthetic')` antes de `python3 scripts/migrate.py update`. O esperado é a recusa da instalação em catálogo preexistente sem transição. O comando retornou 0 e gravou a base. Repetir em outro banco descartável, desta vez criando o enum depois da instalação. A assinatura manteve o mesmo valor, e `python3 scripts/migrate.py rollback-count --count=1` retornou 0. O resultado esperado pela spec é preservar o estado e recusar a reversão na presença do objeto estranho.

A precondition de instalação em `db.changelog-master.yaml` consulta relações e rotinas do schema. A assinatura em `changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql` também deixa tipos independentes fora do inventário. Os dois cenários confirmam a lacuna com um tipo sintético, preservado em ambas as execuções. [Pré-requisitos do plano](../../plan.md#pré-requisitos) · [Spec de migração, papéis e recuperação](../../spec.md#migração-papéis-e-recuperação)

### P3 · Pré-validação aceita a concessão sobre `ctid`

Preparar uma role sintética de consulta com `USAGE` no schema antes da instalação para registrar esse estado na assinatura. Depois da instalação, adicionar somente SELECT sobre `ctid` em `databasechangelog` por concessão administrativa. `has_column_privilege` passou de falso para verdadeiro e o `update` foi aceito com código 0 e histórico inalterado. O esperado pela matriz é recusar a concessão sobre uma coluna de controle a essa role. O filtro `attribute.attnum > 0` em `config/preflight.sql` exclui colunas de sistema. O alcance observado foi o privilégio sobre esse metadado. [Modelo vigente](../../dev-db/modelo-vigente.md#decisões-da-instalação) · [Revisão técnica anterior](../../code-review/f01-story-001.md#revisão-técnica-do-commit-107b5cb)

## Alcance

A validação cobriu PostgreSQL e Liquibase reais em laboratório descartável. A integração com a aplicação, a prontidão e os workflows AWS são verificações das demais entregas da F01. [Spec §7](../../spec.md#7-estratégia-de-testes) · [PRD §9](../../../PRD.md#9-critérios-de-aceitação)
