# RF-STORY-001 — Clean Architecture sem módulos

[Story no Trello](https://trello.com/c/sLATq4Hy/120-story-rf-story-001-refatorar-service-para-clean-architecture-sem-m%C3%B3dulos) · [decisão ADR-001](../adr/ADR-001-backend-clean-architecture.md) · [guideline backend](../guidelines/backend.md).

## Complemento — Domain puro e Services em Application

Solicitação posterior de 2026-11-13: concentrar contratos/comportamento independentes em Domain, sem transferir dependências ASP.NET/EF. Services pertencem exclusivamente a Application. Nenhuma tarefa ou movimentação nova no Trello foi feita para este complemento, conforme orientação expressa; o histórico abaixo descreve a entrega inicial.

| Responsabilidade atual | Localização |
| --- | --- |
| Requests, Responses, Models, Enums, EventAuditBuilder e EventResponseMapper | Domain, usando biblioteca base do .NET |
| EventEntity e cinco POCOs PersistenceEntity | Domain/Entities; configurações EF continuam em Infrastructure |
| Leitura JSON estrita independente de HTTP | Domain/Readers/JsonBodyReader, recebendo Stream/Type/cancelamento |
| Serviços de evento, cursor, telemetria e ator local | Application/Services, sem ASP.NET/EF |
| Proteção criptográfica de cursor | Application/Interfaces/IEventCursorProtector e API/DataProtectors/EventCursorDataProtector |
| Mídia/limites HTTP, MVC, OpenAPI e health | API/Readers/HttpJsonBodyReader, InputFormatters/EventMvcJsonInputFormatter, Transformers/EventOpenApiDocumentTransformer e Writers/HealthHttpResponseWriter |
| Tradução de falhas dos providers | Infrastructure/Mappers/EfPostgresExceptionMapper |

Domain continua sem PackageReference, FrameworkReference ou ProjectReference; Application referencia apenas Domain. Manifests/lockfiles não mudaram. Dois testes de arquitetura verificam as dependências compiladas e a localização das implementações Service. Nomes técnicos explicitam a exceção de camada; não há pastas vazias reservadas.

[Plano e evidências do complemento](../testing/2026-11-13_10-03-49/2-execucao.md). HLD 1.2, ADR-001, AGENTS e guidelines descrevem a distribuição atual. As seções seguintes preservam revisões, números e tarefas da primeira execução.

Backend: [9aa159f43a5a18f36999395488e6967ec376a6e4](https://github.com/diorgenesbackes/scrapbook-service/commit/9aa159f43a5a18f36999395488e6967ec376a6e4). Probes: [1ed8d2f296a06392b6133b7583aaddf6874f7f4f](https://github.com/diorgenesbackes/scrapbook-tests/commit/1ed8d2f296a06392b6133b7583aaddf6874f7f4f). Documentação: este complemento e seu commit na mesma feature.

## Entrega inicial

Solicitação explícita do responsável em 2026-11-13. Backend reorganizado por camadas/conceitos: Controllers MVC, Services, Repositories, Entities, Requests/Responses, mappers, validators e configurações. Todos os tipos declarados possuem arquivo próprio e sufixo do conceito; 83 declarações conferidas. Modules/Endpoints e namespaces funcionais foram removidos, inclusive reservas vazias.

EventController/HealthController são controllers reais. EventService implementa IEventService e depende de IEventRepository/IUnitOfWork; EventRepository/AccountRepository compartilham ScrapBookDbContext scoped. Cinco IEntityTypeConfiguration preservam os mapeamentos utilizados. O domínio continua independente de HTTP/EF/AWS.

| Antes | Agora |
| --- | --- |
| Api/Endpoints/HealthEndpoints | Api/Controllers/HealthController |
| Api/Modules/Events/EventEndpoints | Api/Controllers/EventController |
| Application/Modules/Events/EventService | Application/Services/EventService + Interfaces/IEventService |
| Infrastructure/Modules/Events/Persistence/EventStore | Infrastructure/Repositories/EventRepository |
| EventsDbContext e IdentityDbContext | Infrastructure/DbContexts/ScrapBookDbContext |
| EventRows com quatro tipos no mesmo arquivo | Infrastructure/Entities, um tipo por arquivo |
| EventHttpContracts reunindo DTOs/leitura/validação/mapping | Requests, Responses, Readers, Validators, Mappers e InputFormatters |
| EventRules/EventConfiguration | Domain/Policies/EventPolicy e Entities/EventEntity |

Contratos JSON, rotas, operationIds, erros, regras, autorização, transações, auditoria, cursores e chaves existentes foram validados por regressão. Nomes CLR/componentes de resposta no OpenAPI acompanham os novos tipos Response. Schemas físicos, 47 migrations e pacotes/lockfiles permanecem compatíveis; nenhuma migration ou dependência nova foi adicionada.

## Branch e revisões

feature/RF-STORY-001 é dependente da GE-STORY-001 ainda não promovida. A base desta refatoração é o conteúdo entregue da GE, preservado em sua feature: service 7a7a446, testes 8a32766 e docs cc935f6. A dependência precisa acompanhar a futura integração seletiva; não houve merge de main/hlg nem publicação.

- Backend: [d7159ce0a3182f271c9d76bca1767564beb7353a](https://github.com/diorgenesbackes/scrapbook-service/commit/d7159ce0a3182f271c9d76bca1767564beb7353a).
- Probes/suítes: [57c5cb51e14a118cd1264b19d8d0fcd60137f13f](https://github.com/diorgenesbackes/scrapbook-tests/commit/57c5cb51e14a118cd1264b19d8d0fcd60137f13f).
- Documentação: este documento e seu commit; HLD 1.1, ADR-001, AGENTS e guidelines atualizados. Evidências anteriores da GE continuam históricas.

## Validação

[Plano e execução](../testing/2026-11-13_08-59-05/2-execucao.md), com resultado sanitizado. Build sem avisos/erros; **80 unitários backend**, **14 jornadas Chromium/WebKit**, dez grupos HTTP/PostgreSQL e probe de contexto/repositórios aprovados. H-03: 200 eventos, 10 concorrentes, 60s, **127.014 requisições, zero erros**, p95 lista/detalhe/update **3,366/3,223/11,930ms**.

Um import antigo no probe e o template de rota sem barra inicial do MVC foram corrigidos/retestados. Serviços do usuário foram parados antes da execução, volume preservado; o laboratório descartável encerrou seus próprios processos/container. Portas 5080/5173/55432 foram conferidas fechadas ao terminar. Sem AWS/dispositivos físicos/carga de uploads ou autenticação real. Frontend não foi alterado.

## Tasks e rastreabilidade

Executor técnico: Codex. Fases abaixo usam timestamps do Trello/UTC, sem apontar horas ativas ou esforço exclusivo. Tasks são verificações/entregas de conceitos; commits agrupam alterações dependentes. Fluxo Todo → Doing → Test → To Deploy; reaberturas permanecem no histórico.

| Task | Aceite/entrega | Primeiro Doing UTC | Último Test UTC | To Deploy UTC | Estado |
| --- | --- | --- | --- | --- | --- |
| [RF-001 — Reorganizar entidades, políticas e relógio do domínio](https://trello.com/c/4N2vXPgc/121-rf-001-reorganizar-entidades-pol%C3%ADticas-e-rel%C3%B3gio-do-dom%C3%ADnio) | Domain/Entities, Policies, Clocks, Exceptions, Enums; nomes terminam no conceito. | 2026-11-13T11:50:22.219Z | 2026-11-13T11:58:40.330Z | 2026-11-13T12:18:01.982Z | To Deploy |
| [RF-002 — Separar comandos e modelos de Application por conceito](https://trello.com/c/sfpSstOM/122-rf-002-separar-comandos-e-modelos-de-application-por-conceito) | Um tipo por arquivo, sufixos Command/Model/Enum; nenhuma pasta funcional. | 2026-11-13T11:50:23.383Z | 2026-11-13T11:58:41.930Z | 2026-11-13T12:18:03.757Z | To Deploy |
| [RF-003 — Definir serviços e contratos de repositório](https://trello.com/c/kqkO7n9j/123-rf-003-definir-servi%C3%A7os-e-contratos-de-reposit%C3%B3rio) | IEventService, IEventRepository, IUnitOfWork e EventService em diretórios próprios. | 2026-11-13T11:50:24.456Z | 2026-11-13T11:58:43.271Z | 2026-11-13T12:18:05.295Z | To Deploy |
| [RF-004 — Unificar DbContext e separar configurações EF](https://trello.com/c/vFHXWo8m/124-rf-004-unificar-dbcontext-e-separar-configura%C3%A7%C3%B5es-ef) | ScrapBookDbContext único, cinco configurações e entidades de persistência; schema/grants compatíveis. | 2026-11-13T11:50:26.308Z | 2026-11-13T11:58:44.790Z | 2026-11-13T12:18:06.646Z | To Deploy |
| [RF-005 — Extrair AccountRepository e serviço de ator local](https://trello.com/c/fbDnIyAp/125-rf-005-extrair-accountrepository-e-servi%C3%A7o-de-ator-local) | Conta consultada pelo repositório compartilhando contexto scoped; opt-in e 401 preservados. | 2026-11-13T11:50:27.449Z | 2026-11-13T11:58:46.037Z | 2026-11-13T12:18:08.140Z | To Deploy |
| [RF-006 — Renomear EventRepository e UnitOfWork](https://trello.com/c/8hA8HMXJ/126-rf-006-renomear-eventrepository-e-unitofwork) | Transações, locks, versão, rollback, auditoria e obrigações preservados; DI atualizada. | 2026-11-13T11:50:29.101Z | 2026-11-13T11:58:47.405Z | 2026-11-13T12:18:09.882Z | To Deploy |
| [RF-007 — Separar requests, responses, leitores, validadores e mappers](https://trello.com/c/NCcKHHyu/127-rf-007-separar-requests-responses-leitores-validadores-e-mappers) | Contratos JSON estritos e bigint string preservados; uma classe por arquivo/conceito. | 2026-11-13T11:50:30.984Z | 2026-11-13T11:58:48.898Z | 2026-11-13T12:18:11.461Z | To Deploy |
| [RF-008 — Substituir endpoints de eventos por EventController](https://trello.com/c/R8LJaxkg/128-rf-008-substituir-endpoints-de-eventos-por-eventcontroller) | Cinco ações MVC com rotas, operationIds, erros, Location e orçamento existentes. | 2026-11-13T11:50:32.441Z | 2026-11-13T11:58:50.269Z | 2026-11-13T12:18:13.073Z | To Deploy |
| [RF-009 — Migrar saúde para HealthController](https://trello.com/c/6R7KMILO/129-rf-009-migrar-sa%C3%BAde-para-healthcontroller) | GET /health mantém JSON mínimo, status 200/503, no-store e independência da identidade. | 2026-11-13T11:50:34.140Z | 2026-11-13T11:58:51.748Z | 2026-11-13T12:18:14.685Z | To Deploy |
| [RF-010 — Reorganizar middleware, opções, extensões e telemetria](https://trello.com/c/Pyu7kRUK/130-rf-010-reorganizar-middleware-op%C3%A7%C3%B5es-extens%C3%B5es-e-telemetria) | Sufixos correspondentes, sem Modules/Endpoints; chave de cursor e sinais mantêm compatibilidade. | 2026-11-13T11:50:35.589Z | 2026-11-13T12:05:56.023Z | 2026-11-13T12:18:16.221Z | To Deploy |
| [RF-011 — Adaptar unitários e validar Controllers](https://trello.com/c/zyOzeqct/131-rf-011-adaptar-unit%C3%A1rios-e-validar-controllers) | Namespaces/conceitos novos; cobertura de respostas, políticas, falhas e identidade. | 2026-11-13T11:50:36.849Z | 2026-11-13T11:58:54.245Z | 2026-11-13T12:18:17.812Z | To Deploy |
| [RF-012 — Adaptar probes e suítes manuais](https://trello.com/c/jee9vAAy/132-rf-012-adaptar-probes-e-su%C3%ADtes-manuais) | Inventário do contexto único; integrações HTTP/PostgreSQL/navegador compatíveis. | 2026-11-13T11:50:38.281Z | 2026-11-13T12:00:54.898Z | 2026-11-13T12:18:19.219Z | To Deploy |
| [RF-013 — Atualizar arquitetura e guidelines](https://trello.com/c/0n8YbNhe/133-rf-013-atualizar-arquitetura-e-guidelines) | AGENTS, backend, HLD, princípios e READMEs descrevem estrutura atual e convenção obrigatória; preservar evidência histórica. | 2026-11-13T11:50:41.480Z | 2026-11-13T12:16:11.384Z | 2026-11-13T12:18:20.537Z | To Deploy |
| [RF-014 — Executar regressão funcional e carga e consolidar evidências](https://trello.com/c/etNp93YY/134-rf-014-executar-regress%C3%A3o-funcional-e-carga-e-consolidar-evid%C3%AAncias) | Build/unitários, HTTP/PG, 14 jornadas, H-03; serviços encerrados ao final; relatório preciso, sem deploy. | 2026-11-13T11:50:42.882Z | 2026-11-13T12:05:57.487Z | 2026-11-13T12:18:22.350Z | To Deploy |

A entrega técnica termina em To Deploy. Revisão do responsável, PRs, integração das features dependentes e deploy são etapas posteriores; não se declara aceite humano/publicação. Para reverter, retornar ao código da GE compatível, mantendo banco/volume/chaves. A simulação local continua explícita e limitada ao desenvolvimento.
