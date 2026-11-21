# Plano de validação funcional da F01-STORY-001

Validar o commit integrado `1adeced61e418c3150d3e47d139091835250906c`, disponível na referência local `origin/dev` do Liquibase. A árvore é idêntica à da história em `107b5cb`. A documentação de referência está em `b45b3197cd6fc8552699317fe229b073e50b6cb6`. [Cartão](https://trello.com/c/gSGvlzEZ) · [Spec §6–7](../../spec.md#6-modelo-de-dados) · [Plano, fase 2](../../plan.md#fase-2-banco-e-execução-local) · [PRD §9](../../../PRD.md#9-critérios-de-aceitação)

Executar `python3 tests/test_baseline.py` a partir de uma cópia temporária do commit, em PostgreSQL 18.6 com Liquibase 4.33.0. As verificações complementares reutilizam o runner para criar outro laboratório local. Cada laboratório usa contêiner próprio, porta dinâmica em loopback e massa sintética.

| Cenário | Fonte | Ação e resultado esperado |
| --- | --- | --- |
| CA1.1 · Instalação | Trello CA1, spec §6 | Validar o changelog e gerar a prévia em banco vazio. Aplicar e conferir 24 tabelas mais dois controles, ownership e histórico. |
| CA1.2 · Catálogo | Trello CA1, spec §6 e dicionário | Conferir colunas, tipos, nulabilidade, metadados, controles de lease, PKs, FKs, CHECKs e índices no catálogo. Exercitar unicidade nas tabelas técnicas. |
| CA2.1 · Reaplicação | Trello CA2, spec §6, PRD §9 | Repetir `update` e comparar catálogo e histórico. Exercitar reaplicação e reversão de uma evolução sintética. |
| CA2.2 · Estado incompatível | Trello CA2, spec §6, plano da F01 | Apresentar objeto preexistente, histórico alheio ou drift e exigir recusa diagnosticável. Conferir preservação do estado anterior. Incluir tipo enumerado no schema como variação de objeto alheio. |
| CA2.3 · Falha de changeset | Trello CA2, spec §7, PRD §9 | Injetar falha após o primeiro DDL de um changeset sintético. Verificar reversão transacional, histórico intacto e reexecução da base. |
| CA3.1 · Matriz da aplicação | Trello CA3, spec §6 | Consultar privilégios efetivos por tabela e executar DML com a identidade da aplicação. Exigir recusa de DDL, de assunção da proprietária e de acesso aos controles. |
| CA3.2 · Auditoria | Trello CA3, spec §6 | Inserir com a identidade da aplicação. Verificar recusa de SELECT, UPDATE, DELETE e TRUNCATE para ela. Conferir disponibilidade de leitura na identidade de consulta. |
| CA3.3 · Grant administrativo fora da matriz | Modelo vigente e README do Liquibase | Retestar o achado técnico de grants de coluna de sistema em controle do Liquibase. A pré-validação deve recusar o alvo enquanto o grant existir. |
| CA4.1 · Reversão vazia | Trello CA4, spec §6–7, PRD §9 | Reverter a instalação vazia e reaplicar no mesmo banco. O schema e os controles são preservados. |
| CA4.2 · Proteções de reversão | Trello CA4, spec §6 | Tentar reverter com dados, objeto estranho no schema ou dependência externa. Exigir recusa com diagnóstico e verificar a preservação da base. Usar tabela e tipo enumerado como variações de objeto estranho. |

Aplicação integrada, prontidão, workflow e ambientes AWS, transição da base histórica e regras das Services pertencem a outras entregas. [Plano da F01](../../plan.md) · [PRD §9](../../../PRD.md#9-critérios-de-aceitação)
