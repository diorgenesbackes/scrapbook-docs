# Resultado do reteste de permissões da F01-STORY-001

**Correção de permissões aprovada no laboratório local.** O commit `76d360b64cc31dfc369f47f660ef7ad196decfb6` inclui colunas de sistema na inspeção de privilégios das duas tabelas de controle do Liquibase. O changeset de instalação mantém o mesmo conteúdo da base integrada. [Plano de reteste](1-plano.md) · [Cartão](https://trello.com/c/gSGvlzEZ) · [Revisão técnica anterior](../../code-review/f01-story-001.md#revisão-técnica-do-commit-107b5cb)

O reteste ocorreu em 19/11/2026 com PostgreSQL 18.6, Liquibase 4.33.0 e Java 26.0.1. O runner criou um contêiner descartável por execução, com porta dinâmica em loopback e massa sintética. Usou-se a imagem PostgreSQL fixada no manifesto do Liquibase. A revisão documental de referência foi `105fb4c`.

| Execução | Resultado |
| --- | --- |
| `python3 tests/test_baseline.py BaselineTests.test_reapply_rejects_control_grants_outside_matrix`, com o teste novo e a pré-validação anterior | Código 1 em 13,587 s. O `update` aceitou as duas concessões sobre `ctid` que o teste esperava ver recusadas. |
| `python3 tests/test_baseline.py`, com a correção | Código 0 em 112,210 s. 18 testes aprovados, sem skips. |
| Verificações complementares de aceite, com a correção | Código 1 em 39,869 s. Quatro verificações aprovadas e duas falhas já observadas nos cenários com enum alheio, sem skips. |

## Resultados por cenário

| Cenário | Evidência | Resultado |
| --- | --- | --- |
| CA3 · Controle de histórico | Com SELECT sobre `ctid` de `databasechangelog` concedido a uma role fora da matriz, o `update` terminou na pré-validação com o diagnóstico “Privilégios efetivos incompatíveis nos controles do Liquibase.” | Aprovado |
| CA3 · Controle de lock | O mesmo grant em `databasechangeloglock` produziu a mesma recusa. A suíte também confirmou a recusa de um grant de tabela e de um grant de coluna comum. | Aprovado |
| CA3 · Preservação e recuperação | A suíte comparou o conteúdo completo da linha do histórico, preservou o inventário de tabelas, revogou os grants sintéticos e confirmou que a reaplicação voltou a passar. A matriz DML e a inserção de auditoria pela aplicação também passaram. | Aprovado |
| CA1–CA4 · Regressões da base | Instalação limpa, catálogo de 24 tabelas e dois controles, reaplicação, falha transacional, auditoria e rollback em base vazia passaram. A suíte confirmou as recusas de rollback com dados, tabela alheia ou dependência externa. | Aprovado para os cenários descritos |
| CA2 e CA4 · Enum alheio | O `update` ainda aceitou um tipo enumerado alheio preexistente. O rollback ainda removeu a base vazia quando o tipo foi criado depois da instalação. O enum foi preservado nas duas execuções. | Reprovado |

A decisão de partir de uma instância nova foi exercitada na instalação limpa. A verificação complementar manteve os dois casos com enum para registrar a pendência de proteção de catálogo. O parecer deste ciclo aprova a correção de permissões. O critério da spec que exige recusa do rollback com objeto alheio continua pendente. [Spec de migração, papéis e recuperação](../../spec.md#migração-papéis-e-recuperação)
