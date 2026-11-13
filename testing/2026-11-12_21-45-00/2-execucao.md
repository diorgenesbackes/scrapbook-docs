# Execução — CRUD HTTP de Eventos

Executor: Codex. Validação local em 2026-11-12/13 UTC (2026-11-12 em Brasília). Revisão final do backend: `6bad53c0db33a301918fd28c814b750f12baf3c9`; suítes: `f7bc7f352febf0c8fccd2a297abf98e0f53c3fb0`. Os 25 cartões da implementação HTTP, incluindo GE-086, passaram de Doing para Test antes das chamadas funcionais. GE-087…GE-099 prepararam e validaram o laboratório manual.

## Ambiente e comandos

PostgreSQL 18.6 em container exclusivo, 47 changesets existentes, migrador sem superusuário e runtime DML. SDK 10.0.401; EF 10.0.15; Npgsql/provider 10.0.3; OpenAPI 10.0.15. A suíte não modifica o volume persistente do Compose.

`dotnet test ScrapBook.sln --no-restore` evoluiu de 50 para 69 e, com instrumentação/catálogo de falhas, 77 aprovados; zero falhas/ignorados na execução final. `python3 events_http.py` e depois `python3 journeys.py all` executaram os dez grupos funcionais de HTTP/PostgreSQL.

## Resultado

- Criação 201/Location confirma evento, slot 1 e auditoria. Falha no INSERT de auditoria e falha diferível no commit deixam a contagem original, sem resposta de sucesso.
- Contas sem vínculo não listam/consultam/editam evento alheio; ausente e alheio têm o mesmo 404. Segundo responsável com fixture de convite aceito consulta e salva as duas configurações.
- Unicode 120/121 e 2.000/2.001, NUL/surrogate, campos indevidos, offsets, body excedido, media type e bigint foram exercitados.
- Edições concorrentes da mesma versão: uma 200, outra 409. No-op não incrementa; versão/estado finais são conferidos no banco. NULL restaura boas-vindas.
- Relógio real com evento encerrado confirma scheduled_locked, transição única immediate_once e preservação do congelamento ao mover data.
- Pending/requested obsoletos são cancelados no commit; dispatched é preservado. Falha posterior de auditoria desfaz evento e cancelamento juntos.
- Paginação 20/default, limites 1–50, cursor inválido/longo/de outra conta e restart de processo com chaves persistidas passaram. TTL expirado/adulteração/versão de payload são cobertos em unitários.
- OpenAPI publica as cinco operações e schemas estritos, bigint string, nulabilidade e campos obrigatórios.
- Flag inativa/ator inválido →401; health →200; ativação em Production impede startup; Origin externo →403; preflight explícito →204.

## Falhas e retestes

O metadata Accepts produzia 415 vazio antes do handler. GE-051 voltou para Doing; a fronteira passou a validar headers antes do delegate e o reteste aprovou Problem Details correlacionado. A interrupção real de conexão mostrou InvalidOperationException do EF encapsulando NpgsqlException; GE-019/049 foram reabertas, passaram a tratar o tipo/SQLSTATE conhecido e o reteste retornou 503 sem SQL/stack/credenciais. Não houve retry da transação.

A revisão final do OpenAPI removeu nulabilidade indevida de expectedVersion/openingMode; runtime continua rejeitando NULL nesses campos. Os limites e resultados finais estão também no [ciclo integrado](../2026-11-12_22-23-17/2-execucao.md).

## Limites

Nenhum consumidor Records, convite de produto, envio, Notifications ou AWS foi implementado. As falhas injetadas e snapshot travado existem somente no banco descartável. O minuto real de 12h não foi aguardado; fronteiras exatas são unitárias, com complemento temporal no PostgreSQL real. Não houve deploy.
