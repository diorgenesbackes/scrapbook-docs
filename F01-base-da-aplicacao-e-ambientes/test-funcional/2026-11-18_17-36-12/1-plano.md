# Plano de validação funcional da F01-STORY-001

**Entrega avaliada**. Instalar e reverter a base PostgreSQL com Liquibase. Liquibase `107b5cb`, branch `feature/F01-STORY-001`, com cópia de trabalho limpa. Documentação `e42254e`, PRD local com blob `5977d455` e spec local com blob `d1a53298`. [Cartão da história](https://trello.com/c/gSGvlzEZ) · [Spec §6 e §7](../../spec.md#6-modelo-de-dados) · [Plano da F01](../../plan.md) · [PRD §9](../../../PRD.md#9-critérios-de-aceitação)

A decisão vigente para esta instalação é um schema `scrapbook`, com 21 tabelas de domínio e operação, três técnicas compartilhadas e dois controles do Liquibase. O inventário histórico de quatro schemas é referência para o conteúdo das tabelas, com os deltas da spec atual. [Dicionário físico histórico](../../../db/DB-Modelo-Fisico.md) · [Modelo vigente](../../dev-db/modelo-vigente.md)

## Cenários e resultados esperados

| Cenário | Fonte | Ator, pré-condição e ação | Resultado esperado e evidência |
| --- | --- | --- | --- |
| CA1.1 | Trello CA1, spec §6 | Pessoa operadora prepara um banco novo, valida o changelog, gera a prévia e executa o update. | Prévia sem criar tabelas de aplicação. Após o update, 24 tabelas exatas e dois controles em `scrapbook`, com proprietário e histórico esperados. Consultar catálogo e histórico. |
| CA1.2 | Trello CA1, spec §6 e dicionário | Pessoa operadora confronta o catálogo instalado com o inventário e os deltas da spec. | Campos, tipos, nulabilidade, defaults dos metadados e do lease, PKs, FKs, CHECKs e índices compatíveis. Confirmar a deduplicação compartilhada com operações sintéticas. |
| CA2.1 | Trello CA2, spec §6, PRD §9 F01 | Pessoa operadora repete a aplicação sobre instalação íntegra. | Mesma assinatura e histórico, sem executar o changeset novamente. |
| CA2.2 | Trello CA2, spec §6, plano da F01 | Pessoa operadora tenta aplicar com objeto incompatível, catálogo alterado ou histórico alheio. | Execução recusada com diagnóstico. Objetos, registro anterior e estado preexistente preservados. |
| CA2.3 | Trello CA2, spec §7, PRD §9 F01 | Pessoa operadora executa changeset sintético que falha após o primeiro DDL em banco instalado. | Falha diagnosticável, histórico anterior intacto, DDL intermediário revertido e reaplicação da base possível. |
| CA3.1 | Trello CA3, spec §6 | Identidade da aplicação autentica no banco preparado e exerce a matriz DML. | Operações declaradas disponíveis nas 24 tabelas, incluindo geração de ID. DDL, concessão de privilégios, assunção da proprietária e leitura dos controles são recusados. |
| CA3.2 | Trello CA3, spec §6 | Aplicação insere auditoria e tenta ler, editar, excluir ou truncar. A role de consulta inspeciona o histórico de auditoria. | Inserção autorizada, mutações recusadas e leitura restrita à role de consulta. Conferir a matriz efetiva e o registro preservado. |
| CA4.1 | Trello CA4, spec §6 e §7, PRD §9 F01 | Pessoa operadora reverte a base vazia com o Liquibase e reaplica. | As 24 tabelas são removidas, os dois controles e o schema são preservados, e a nova instalação retorna ao inventário e histórico esperados. |
| CA4.2 | Trello CA4, spec §6 | Pessoa operadora tenta reverter separadamente com uma linha existente, um objeto alheio no schema ou uma dependência externa. | Cada tentativa é recusada, preservando dados, objetos, catálogo e histórico conforme o cenário. |

## Ambiente e alcance

Executar a suíte versionada `python3 tests/test_baseline.py` em `scrapbook-liquibase`. Complementar as asserções com consultas ao catálogo, à matriz efetiva e ao estado após as recusas. Usar PostgreSQL 18.6 em Docker, Liquibase 4.33.0 e dados sintéticos. Cada execução prepara seu próprio contêiner em loopback, com bancos por cenário e remoção do contêiner ao término. A execução exercita PostgreSQL e Liquibase reais.

O aceite desta história cobre o banco inicial em laboratório. UI e prontidão integrada, aplicação e rollback por workflow AWS, isolamento de ambientes, observabilidade do deploy com `executionId`, transição de dados anteriores e regras das Services pertencem às entregas correspondentes da F01 e features seguintes. [Spec §7](../../spec.md#7-estratégia-de-testes) · [Plano da F01](../../plan.md)
