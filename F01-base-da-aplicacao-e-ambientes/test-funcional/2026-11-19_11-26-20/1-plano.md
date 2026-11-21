# Plano de reteste da F01-STORY-001

Validar a correção do CA3 para concessões administrativas sobre colunas de sistema dos controles do Liquibase. A base da correção é `107b5cb` na branch `feature/F01-STORY-001`. O provisionamento de referência começa em PostgreSQL novo e vazio. [Cartão](https://trello.com/c/gSGvlzEZ) · [Spec §6–7](../../spec.md#6-modelo-de-dados) · [PRD §9](../../../PRD.md#9-critérios-de-aceitação)

| Cenário | Ação e resultado esperado |
| --- | --- |
| CA3 · Controle de histórico | Provisionar uma role sintética fora da matriz com `USAGE` no schema antes da instalação. Depois do primeiro `update`, conceder SELECT sobre `ctid` em `databasechangelog`. O próximo `update` deve falhar na pré-validação com diagnóstico de permissões. |
| CA3 · Controle de lock | Repetir o mesmo cenário em `databasechangeloglock` e exigir o mesmo diagnóstico. |
| CA3 · Preservação e recuperação | Conferir inventário e conteúdo do histórico após as recusas. Revogar as concessões sintéticas e reaplicar com a mesma credencial migradora. O `update` deve passar e preservar o histórico. |
| CA1–CA4 · Regressões da base | Rodar a suíte completa em instância descartável. Repetir as verificações complementares de catálogo, permissões, rollback e objetos alheios, com dados sintéticos. |

Executar o teste direcionado antes da correção para confirmar que ele detecta a falha. Executar a suíte e as verificações complementares sobre o candidato corrigido, em PostgreSQL 18.6 e Liquibase 4.33.0 com contêineres e portas dinâmicas em loopback.
