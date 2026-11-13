# Plano — contratos puros e adaptadores explícitos

Ajuste solicitado pelo responsável em 2026-11-13, complementar à RF-STORY-001, na mesma feature. Sem novas tarefas ou movimentações no Trello, conforme orientação expressa. Domain não receberá dependências ASP.NET, EF Core, Npgsql ou referências às demais camadas.

Verificar contratos Requests/Responses, Models, Builders, Enums, Entities POCO, mapper de resposta e leitor JSON em Domain. Services exclusivamente em Application. Conferir separação entre EventCursorService e EventCursorDataProtector, JSON puro e HttpJsonBodyReader; formatter MVC, transformer OpenAPI, writer HTTP e mapper EF/PostgreSQL permanecem nas camadas técnicas, com nomes explícitos. Manter pacotes, lockfiles, rotas, payloads, propósito criptográfico e banco existentes.

Executar restore locked, build/unitários (incluindo regras de dependência/localização dos Services), build dos probes e `python3 journeys.py all` em scrapbook-tests. A suíte cobre contexto único/cinco mapeamentos, dez grupos HTTP/PostgreSQL, 14 jornadas Chromium/WebKit, falhas/privacidade e H-03: 200 eventos, 10 concorrentes, 60s após aquecimento, zero erros e p95 ≤500ms por operação.

Usar somente laboratório descartável. Preservar serviços do usuário parados, volume e evidências históricas. Registrar resultados e limites em 2-execucao.md; sem implantação.
