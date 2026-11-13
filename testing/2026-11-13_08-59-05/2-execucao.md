# Execução — refatoração sem módulos

Executor: Codex. Branch `feature/RF-STORY-001`, dependente da GE-STORY-001. Execução integrada de 2026-11-13, **09:06:27–09:07:54 de Brasília**, em laboratório descartável; não houve deploy.

## Alteração e verificações

- Controllers MVC reais: EventController e HealthController; AddControllers/MapControllers e rotas por atributos. Input formatter mantém JSON estrito e Problem Details.
- Diretórios por conceito, sem Modules/Endpoints ou divisões funcionais; 83 tipos declarados conferidos, um por arquivo, com nome/sufixo correspondente. Program.cs é a entrada convencional.
- ScrapBookDbContext único/scoped, cinco entidades utilizadas e cinco configurações EF. Repositórios compartilham contexto; EventService depende de IEventService/IEventRepository/IUnitOfWork conforme o papel de cada implementação.
- Pacotes/lockfiles e SQL físico não foram alterados. Banco de teste aplica os mesmos 47 changesets, separados do Compose/volume do usuário.

## Comandos e resultado

`dotnet build ScrapBook.sln --no-restore`, `dotnet test ScrapBook.sln --no-restore` e build dos probes aprovados. **80 unitários backend, zero falhas/ignorados**; três novos cenários de HealthController verificam 200/up, degraded/unhealthy →503/down, corpo mínimo e no-store. A formatação foi conferida por dotnet format whitespace e IDE0005.

`python3 journeys.py all` aprovado: contexto único/rollback, dez grupos HTTP/PostgreSQL, **14 jornadas Chromium/WebKit**, indisponibilidade/orçamento/privacidade e H-03. Chromium 158.0.8127.18 em 1280×720/Tokyo; WebKit 27.2 em iPhone 13 emulado 390×664/Los Angeles; ambos também em 320×740. Contratos, independência de estrutura/abertura, conflito, resposta perdida, cursor, bigints/datas e proxy/health mantiveram comportamento validado.

Runtime indisponível retornou 503 em **3,578ms**; Identity + lock de Events esgotaram o orçamento compartilhado em **5.013,618ms**, sem nova gravação. Conferidas **163 linhas JSON**, sem marcadores de nome/mensagem/credencial; auditoria manteve allowlist. Nenhum exportador AWS foi criado.

## Carga H-03

Apple M4 Pro/arm64, 12 CPUs lógicas, macOS 26.6.2, HTTP loopback e PostgreSQL 18.6 no Docker Desktop; SDK 10.0.401, Python 3.13.15, Playwright 1.65.0. Uma conta, **200 eventos**, **10 concorrentes**, três ciclos de aquecimento por worker e **60s medidos**.

| Operação | Requisições | Erros | p95 | Meta |
| --- | ---: | ---: | ---: | ---: |
| Lista | 42.338 | 0 | 3,366ms | ≤500ms |
| Detalhe | 42.338 | 0 | 3,223ms | ≤500ms |
| Atualização | 42.338 | 0 | 11,930ms | ≤500ms |

Total: 127.014 requisições medidas, sem aquecimento/preparação. Atualizações usam versões válidas em eventos próprios por worker. [Resultado sanitizado](resultado.json).

## Falhas, correções e retestes

Um import antigo do probe referenciava namespace de doubles de UnitTests após a renomeação; removido, build e probe real aprovados. Na primeira regressão MVC, HTTP e 14 jornadas passaram, mas a checagem de telemetria detectou RoutePattern sem barra inicial. RF-010/014 voltaram a Doing; normalização do template foi ajustada e a execução completa acima aprovou. Aviso NO_COLOR/FORCE_COLOR não alterou os resultados.

## Limites e encerramento

Emulação não substitui aparelhos físicos, matriz completa de versões ou leitor de tela. Fronteiras exatas de 12h permanecem unitárias, complementadas por PostgreSQL com relógio real e eventos encerrados. Sem autenticação real, AWS, upload/galeria, disponibilidade global ou RTO/RPO medidos. Histórico da GE foi preservado e referenciado como versão anterior da organização do código.

O laboratório removeu seu container/processos. Frontend, API e PostgreSQL do usuário, parados a pedido no início, permanecem parados; volume preservado. RF segue para revisão em To Deploy, sem merge/publicação.
