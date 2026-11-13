# Execução — Domain puro e Services em Application

Executor: Codex. Complemento da RF-STORY-001 em `feature/RF-STORY-001`. Laboratório descartável, 2026-11-13T10:05:43.476683-03:00 a 2026-11-13T10:07:12.406269-03:00. Sem novas tarefas/movimentações no Trello, conforme solicitação do responsável. Sem deploy.

Backend: [9aa159f43a5a18f36999395488e6967ec376a6e4](https://github.com/diorgenesbackes/scrapbook-service/commit/9aa159f43a5a18f36999395488e6967ec376a6e4). Probes: [1ed8d2f296a06392b6133b7583aaddf6874f7f4f](https://github.com/diorgenesbackes/scrapbook-tests/commit/1ed8d2f296a06392b6133b7583aaddf6874f7f4f). Documentação: este complemento e seu commit na mesma feature.

## Estrutura e dependências

Domain concentra Requests/Responses, Models, Enums, Builders, Entities POCO, EventResponseMapper e JsonBodyReader independente de HTTP. Não possui PackageReference, FrameworkReference ou ProjectReference. Application referencia somente Domain; ambos os assemblies usam somente biblioteca base do .NET além dessa referência interna. Os quatro Services pertencem exclusivamente a Application. 87 declarações de tipos conferidas, uma por arquivo/conceito.

EventCursorService usa IEventCursorProtector, implementado por EventCursorDataProtector na API. Propósito Events.List.v1, aplicação/chaves, expiração de 24h, payload e vínculo de conta preservados. HttpJsonBodyReader mantém validação de mídia/tamanho e tradução de erros; EventMvcJsonInputFormatter, HealthHttpResponseWriter, EventOpenApiDocumentTransformer e EfPostgresExceptionMapper explicitam as dependências técnicas nas camadas externas. Configurações EF e DbContext continuam na Infrastructure, mapeando cinco POCOs.

Manifests, referências de projetos, versões e lockfiles permanecem inalterados. Nenhuma migration ou alteração do banco persistente; mesmas 47 migrations no laboratório. UI e infraestrutura não foram alteradas.

## Verificações

- Restore locked da solution/probes e build dos probes: aprovados, zero avisos/erros.
- `dotnet test ScrapBook.sln --no-restore`: **82 unitários aprovados, zero falhas/ignorados**, incluindo dois novos testes de arquitetura para dependências e localização dos Services. Demais cenários de JSON/cursor/health e regras foram adaptados às extrações.
- `dotnet format whitespace` e diagnóstico IDE0005 executados; revisão de imports técnicos e ownership de conceitos aprovada.
- `python3 journeys.py all`: contexto scoped único, cinco mapeamentos, rollback/repositórios, dez grupos HTTP/PostgreSQL e **14 jornadas Chromium/WebKit** aprovados. OpenAPI, bigints/datas, JSON estrito, autorização, transações/concorrência, cursores e health mantiveram seus contratos.
- Indisponibilidade retornou 503 em **3.254ms**. Orçamento compartilhado Identity + lock de Events: **4978.406ms**, sem nova gravação. 163 linhas estruturadas verificadas, sem marcadores privados; auditoria manteve allowlist.

## Carga H-03

Mesma máquina local arm64/12 CPUs, macOS 26.6.2 e PostgreSQL 18.6/Docker Desktop. 200 eventos, 10 concorrentes, três ciclos de aquecimento por worker e 60s medidos. HTTP/DB loopback; não representa infraestrutura AWS.

| Operação | Requisições | Erros | p95 | Meta |
| --- | ---: | ---: | ---: | ---: |
| Lista | 42877 | 0 | 3.321ms | ≤500ms |
| Detalhe | 42877 | 0 | 3.187ms | ≤500ms |
| Atualização | 42877 | 0 | 12.026ms | ≤500ms |

Total: **128631 requisições**, zero erros. [Resultado sanitizado](resultado.json).

## Correções, limites e encerramento

O primeiro build encontrou duas diretivas using adicionadas após declarações; foram reposicionadas e build/unitários aprovados antes da regressão integrada. Aviso NO_COLOR/FORCE_COLOR no Playwright não afetou resultados.

Emulação de Chromium desktop/Tokyo e WebKit móvel/Los Angeles não equivale a aparelhos físicos ou auditoria completa de acessibilidade. Não mede AWS, autenticação real, uploads, disponibilidade global nem recuperação de desastre. Evidências anteriores permanecem históricas.

O laboratório encerrou seus processos e removeu seu container exclusivo. Frontend, API e PostgreSQL do usuário permanecem parados, com volume preservado. Sem promoção de branch ou publicação.
