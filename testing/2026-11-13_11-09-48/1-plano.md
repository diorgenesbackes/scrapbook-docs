# Plano — dockerizar os projetos

Cartão [dockerizar os projetos](https://trello.com/c/NCkBjG4R), branch feature/6 nos cinco repositórios afetados, dependente das features GE-STORY-001/RF-STORY-001 ainda não promovidas. Dockerfiles de API/UI, rede bridge por nomes de serviço, proxy UI→service:8080 e conexão service→postgres:5432. Portas publicadas apenas em loopback; volume existente do usuário preservado.

Validar builds multi-stage com imagens base fixadas por digest, restore locked/npm ci, contexts sem credenciais/bibliotecas locais, runtime sem SDK/Node, usuários não root, healthchecks e leitura somente nos filesystems de aplicação. Executar unitários .NET e npm run check; conferir opt-in de identidade somente Development e container.

Executar manualmente python3 docker_stack.py em scrapbook-tests: Compose com nome, credenciais, portas, rede e volumes exclusivos; 47 migrations, runtime DML, identidade desabilitada por padrão, health direto/proxy, SPA/assets/404, CRUD JSON/versão/auditoria via proxy, DNS interno, persistência de evento e cursor após down/up. Exercitar helpers locais no contexto exclusivo. Rodar 14 jornadas Chromium/WebKit e H-03 com 200 eventos/10 concorrentes/60s, zero erros e p95 ≤500ms por operação através de Nginx/API/PostgreSQL. Registrar imagens e resultados sanitizados.

Cleanup somente do projeto descartável; manter banco, volume e serviços do usuário no estado inicial parado. Não publicar imagens, integrar main/hlg ou implantar AWS. Mover tarefas para Test antes da execução funcional e To Deploy depois de aprovadas, com transições e evidências reais.
