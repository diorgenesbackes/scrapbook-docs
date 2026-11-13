# Plano — refatoração Clean Architecture sem módulos

Solicitação de 2026-11-13: estrutura por conceito/sufixo, Controllers MVC e DbContext único em scrapbook-service. Branch feature/RF-STORY-001, dependente da GE-STORY-001 não promovida. Frontend/API/PostgreSQL do usuário foram parados antes da refatoração, preservando volume.

Verificar build/restore locked e unitários; cada classe/record em arquivo próprio, namespace por camada/conceito, sufixo correspondente, ausência de Modules/Endpoints. Conferir EventController/HealthController reais, IEventService/IEventRepository e cinco mapeamentos no ScrapBookDbContext scoped, com configurações separadas.

Executar `python3 journeys.py all` em scrapbook-tests: probe do contexto/repositórios; HTTP/PostgreSQL (autorizações, CRUD, JSON, rollback, concorrência, gatilhos, cursor/restart e OpenAPI); 14 jornadas Chromium/WebKit; indisponibilidade/orçamento/privacidade; H-03 com 200 eventos, 10 concorrentes, 60s após aquecimento, p95 ≤500ms por operação e zero erros. Usar banco/container exclusivo e fixture sintética. A suíte limpa seus processos/container; manter serviços do usuário parados ao terminar.

Conferir documentação atual, links e referências de namespaces, preservando evidências históricas da GE. Falhas reabrem Doing antes da correção; tarefas aprovadas vão a To Deploy. Não promover ambiente nem implantar.
