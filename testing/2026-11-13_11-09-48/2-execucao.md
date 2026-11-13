# Execução — dockerizar os projetos

Cartão [dockerizar os projetos](https://trello.com/c/NCkBjG4R), executor Codex, branch feature/6. Execução integrada aprovada: 2026-11-13T11:21:28.604694-03:00 a 2026-11-13T11:23:26.604263-03:00. Docker Desktop 29.8.2, Compose 5.6.0, arm64/12 CPUs, macOS 26.6.2. Sem deploy ou publicação de imagens.

## Entrega e operação

- Dockerfile multi-stage da API: SDK 10.0.401, restore locked, publish Release, runtime ASP.NET 10.0.12/noble sem SDK; app/1654, porta 8080 e healthcheck HTTP.
- Dockerfile multi-stage da UI: Node 26.13.0/npm 11.20.0, npm ci/build, somente dist em Nginx unprivileged 1.30.2; sem Node/node_modules no runtime. Contextos com allowlist e exclusão de credenciais/dependências locais; imagens base fixadas por digest.
- Compose da infraestrutura com rede bridge scrapbook: UI encaminha /api e /health a service:8080; API usa postgres:5432 com credencial DML distinta do administrador/migrador. A UI não recebe credenciais. Publicações somente em loopback: UI 5173, API 5080 e banco 55432, configuráveis.
- API/UI sem root, filesystem somente leitura, /tmp temporário, capabilities retiradas e no-new-privileges. Health da API mede vitalidade, independente do banco; /ui-health mede Nginx. Depends_on aguarda saúde na ordem PostgreSQL→API→UI.
- SPA entrega index em rotas; assets/API ausentes retornam 404. Proxy preserva Problem Details/query/status/headers e não repete escritas. Upstream com pool keepalive 32/worker, HTTP 1.1 e DNS renovável; [referência Nginx](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive).
- postgres_data conserva o volume existente; cursor_keys conserva as chaves entre recriações Docker. O diretório de chaves do modo nativo é separado: cursores emitidos por esse modo requerem suas chaves originais; não apagar/importar chaves automaticamente.
- stack_local.py oferece build/up/stop/down/status. A preparação continua explícita: init, subir postgres, Liquibase update, provision-runtime e bootstrap-actor. Up sem flag usa Production/identidade desativada; --enable-development-identity usa fixture real em Development, com AllowContainerBinding e indicação de container. Sem identidade real de produção, migrations ou seed no startup.

[Comandos operacionais](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/feature/6/README.md). Rede por nomes/portas de container segue o [Docker Compose](https://docs.docker.com/compose/how-tos/networking/); a bridge local não é uma política de isolamento de saída de produção.

## Verificações e resultados

`docker compose build service ui` aprovado a partir dos manifests/lockfiles, sem usar bibliotecas do host. `dotnet test ScrapBook.sln --no-restore`: **86 unitários**, zero falhas/ignorados; quatro novos casos verificam o opt-in/container/Development. `npm run check`: lint, build e **39 unitários** aprovados. `py_compile` dos helpers/suíte e diff --check aprovados. stack_local.py status confirmou o projeto persistente parado.

`python3 docker_stack.py` aprovado em projeto Compose exclusivo, portas aleatórias, três senhas separadas e volumes próprios. Aplicou as mesmas **47 migrations/33 tabelas**, preparou grants DML/fixture sintética e confirmou:

- Health direto e via UI; Events retorna 401 antes do opt-in e funciona depois dele.
- DNS UI→service→postgres, runtime sem SDK/Node, usuários não root, saúde e filesystem somente leitura.
- CRUD pelo proxy, versão otimista/409, JSON estrito/413 com Problem Details, membro e auditoria no banco; mesma representação persistida na API direta.
- SPA/assets e 404 adequados. Dados e cursor válidos após down/up; tabelas/histórico preservados.
- **14 jornadas** Chromium desktop/Tokyo e WebKit móvel emulado/Los Angeles; mesmos cenários de datas, conflito, resposta perdida, estrutura/abertura, paginação e health.

## Carga H-03

200 eventos, 10 concorrentes, três ciclos de aquecimento por worker e 60s medidos, através de host→Nginx→API→PostgreSQL. Uma conta e eventos próprios por worker, com versões válidas.

| Operação | Requisições | Erros | p95 | Meta |
| --- | ---: | ---: | ---: | ---: |
| Lista | 44065 | 0 | 4.947ms | ≤500ms |
| Detalhe | 44065 | 0 | 4.546ms | ≤500ms |
| Atualização | 44065 | 0 | 10.02ms | ≤500ms |

Total: **132195 requisições medidas, zero erros**. [Resultado sanitizado](resultado.json), incluindo identificadores das imagens locais avaliadas. Não são digests de publicação ECR.

## Falhas corrigidas e limites

Na primeira execução, a consulta do novo teste usava event_id em vez dos nomes compostos físicos das FKs; corrigida. Depois, duas jornadas de paginação falharam por massa com menos de 20 eventos; a fixture passou a preparar 25 registros pelo HTTP. A primeira carga não atingiu o critério H-03; a configuração do proxy foi ajustada para upstream com pool persistente/DNS dinâmico, e a suíte passou a imprimir métricas agregadas antes de aplicar os critérios. Os números dessa tentativa anterior não foram preservados; não se atribui uma causa específica sem diagnóstico suficiente. A execução completa acima aprovou após os ajustes, com o mesmo perfil/meta. Aviso NO_COLOR/FORCE_COLOR não afetou o Playwright.

Logs privados de comandos/containers ficam apenas em artifacts, ignorados, sem credenciais no relatório versionado. Nenhuma autenticação real, AWS, carga de uploads, aparelho físico, disponibilidade global ou RTO/RPO foi validado. A stack é local; destinos Amplify/ECS continuam os do HLD. O teste não comprova builds amd64, apenas a arquitetura arm64 executada.

## Dependências Git

Feature/6 parte das revisões locais já entregues das features anteriores; não carrega branches de ambiente. GE-STORY-001/RF-STORY-001 ainda não foram promovidas. Promover suas dependências antes de integrar a dockerização ou tratar o conjunto como entrega explicitamente validada. Liquibase main não foi alterado.

| Repositório | Base | Feature de origem |
| --- | --- | --- |
| scrapbook-service | 9aa159f43a5a18f36999395488e6967ec376a6e4 | feature/RF-STORY-001 |
| scrapbook-ui | fd473d8c1a0e9e632eb8cb1e4366c53f0ee29f83 | feature/GE-STORY-001 |
| scrapbook-infrastructure | f2a249e79f1cf57b05ba4be9d0c173571b9c1bf2 | feature/GE-STORY-001 |
| scrapbook-tests | 1ed8d2f296a06392b6133b7583aaddf6874f7f4f | feature/RF-STORY-001 |
| scrapbook-docs | 3e62a4de3f41f730c4019e651f49bb8f6978aae3 | feature/RF-STORY-001 |

## Revisões entregues

- scrapbook-service: [d0ba999f725acfd2c302b522f1f0e05ed49dabda](https://github.com/diorgenesbackes/scrapbook-service/commit/d0ba999f725acfd2c302b522f1f0e05ed49dabda).
- scrapbook-ui: [6f270d9dbeee085fe36d02ef84ea86d2ff881ed6](https://github.com/diorgenesbackes/scrapbook-ui/commit/6f270d9dbeee085fe36d02ef84ea86d2ff881ed6).
- scrapbook-infrastructure: [51c1408a5ef9a829035ae54ac0767d2ad52f5849](https://github.com/diorgenesbackes/scrapbook-infrastructure/commit/51c1408a5ef9a829035ae54ac0767d2ad52f5849).
- scrapbook-tests: [b8c8f4a2b2b51f469bf8a2c56774bae8d3b08092](https://github.com/diorgenesbackes/scrapbook-tests/commit/b8c8f4a2b2b51f469bf8a2c56774bae8d3b08092).
- scrapbook-docs: este relatório e seu commit em feature/6; HLD 1.3, guidelines e índice atualizados.

## Encerramento e recuperação

O laboratório removeu somente seus containers, rede e volumes de projeto aleatório. Portas 5080/5173/55432 e Docker foram conferidos sem serviços em execução; o banco/volume persistentes não foram iniciados ou modificados. A parada habitual usa stack_local.py stop/down e preserva dados/chaves; nunca down -v no projeto persistente. Para retirar os containers de aplicação, parar a stack e retornar ao modo nativo/revisões anteriores compatíveis, preservando banco e chaves; não reverter migrations para retirar a dockerização.

## Rastreabilidade

Transições reais do Trello/UTC, sem horas estimadas ou esforço exclusivo. Tasks planejadas em Todo, iniciadas em Doing, testadas em Test e aprovadas em To Deploy. Reaberturas dos testes/configuração permanecem no histórico e no arquivo de transições.

| Task | Todo UTC | Primeiro Doing UTC | Último Test UTC | To Deploy UTC |
| --- | --- | --- | --- | --- |
| [DK-001 — Criar imagem Docker da API](https://trello.com/c/tQVRF3AP/146-dk-001-criar-imagem-docker-da-api) | 2026-11-13T14:04:21.092Z | 2026-11-13T14:04:32.775Z | 2026-11-13T14:11:07.387Z | 2026-11-13T14:26:58.909Z |
| [DK-002 — Criar imagem Docker da UI e proxy HTTP](https://trello.com/c/naZpNawi/147-dk-002-criar-imagem-docker-da-ui-e-proxy-http) | 2026-11-13T14:04:22.629Z | 2026-11-13T14:06:14.546Z | 2026-11-13T14:20:57.237Z | 2026-11-13T14:27:01.911Z |
| [DK-003 — Integrar API, UI e PostgreSQL no Compose](https://trello.com/c/iLUJvfIk/148-dk-003-integrar-api-ui-e-postgresql-no-compose) | 2026-11-13T14:04:24.123Z | 2026-11-13T14:06:16.034Z | 2026-11-13T14:11:10.923Z | 2026-11-13T14:27:05.200Z |
| [DK-004 — Adaptar identidade local e operação da stack](https://trello.com/c/IBrMM5bt/149-dk-004-adaptar-identidade-local-e-opera%C3%A7%C3%A3o-da-stack) | 2026-11-13T14:04:25.302Z | 2026-11-13T14:06:17.343Z | 2026-11-13T14:11:12.478Z | 2026-11-13T14:27:07.390Z |
| [DK-005 — Validar imagens e integração funcional](https://trello.com/c/zeOYKmAw/150-dk-005-validar-imagens-e-integra%C3%A7%C3%A3o-funcional) | 2026-11-13T14:04:26.379Z | 2026-11-13T14:09:18.463Z | 2026-11-13T14:14:03.083Z | 2026-11-13T14:27:09.857Z |
| [DK-006 — Validar persistência, navegador e carga em containers](https://trello.com/c/NLOsfLyp/151-dk-006-validar-persist%C3%AAncia-navegador-e-carga-em-containers) | 2026-11-13T14:04:27.875Z | 2026-11-13T14:09:19.708Z | 2026-11-13T14:20:58.939Z | 2026-11-13T14:27:13.077Z |
| [DK-007 — Documentar stack e evidências da entrega](https://trello.com/c/xtcPY7mK/152-dk-007-documentar-stack-e-evid%C3%AAncias-da-entrega) | 2026-11-13T14:04:29.888Z | 2026-11-13T14:09:21.256Z | 2026-11-13T14:18:47.814Z | 2026-11-13T14:27:15.698Z |

[Transições completas](transicoes.json). Cartão e sete tasks em To Deploy; aguardam revisão/promoção, sem declarar aceite humano ou deploy.
