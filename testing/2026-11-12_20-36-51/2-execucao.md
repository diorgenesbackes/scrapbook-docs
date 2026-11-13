# Execução — persistência e regras de Events

Data: 2026-11-12. Início da fase Test: 20:36:51 de Brasília. Executor: Codex. Service: `3733180`; probes: `5b95c7e`. Ambiente: PostgreSQL 18.6 local, SDK 10.0.401; código em `feature/GE-STORY-001`.

## Validação

Build sem warnings/erros. Os unitários anteriores aos testes adicionais passaram com 38 casos; o conjunto final teve 50 aprovados, zero falhas e zero ignorados.

`python3 run.py models` foi executado contra PostgreSQL real e confirmou:

- DbContext scoped, quatro entidades exclusivamente em Events e Identity consultado pelo próprio contrato/contexto.
- Nome com 120 e mensagem com 2.000 emojis persistidos e lidos integralmente, sem truncamento UTF-16 no provider.
- Evento, membro e auditoria aceitos pelos grants e constraints; rollback deixou a contagem original preservada.
- Filtro por vínculo aceitou o ator e recusou conta sem vínculo.
- Triggers retornaram version/access_version finais; SaveChanges sem alterações não escreveu.

Uma primeira execução falhou com SQLSTATE 42501 porque o EF incluía `events_covers_id` no INSERT. GE-010 retornou para Doing, o campo foi marcado como somente leitura e o reteste aprovou sem ampliar os grants.

HTTP: Origin externo recebeu 403 com `REQUEST_ORIGIN_NOT_ALLOWED`, UUID/correlação e sem cache; preflight da origem explícita recebeu 204. Processo com identidade habilitada em Production recusou inicialização antes de servir requisições.

Os unitários cobrem antes/no/depois de 12h e tolerância até 13h com tempo controlado. O probe usa o relógio real do PostgreSQL; não afirma ter alterado `clock_timestamp()` ou executado exatamente ao meio-dia do host.

## Limites

Ainda não há CRUD HTTP exposto nesta leva. Paginação/cursor HTTP, casos de uso, endpoints, concorrência via HTTP, carga H-03 e telas serão validados nas próximas levas. O teste de abertura com slots não cria fluxo de convite; não há dados de Records ou Notifications consultados por Events. Nenhum deploy realizado.
