# Plano: PP-STORY-001

Plano consolidado a partir das 32 subtarefas e do contrato definido antes da implementação. [FDD](../../fdd/FDD-Acesso-Publico-do-Evento/FDD-Acesso-Publico-do-Evento.md).

## Escopo e critérios

Validar AC01–AC05: acesso anônimo correto, preservação do código, SPA com acesso direto/reload/nova aba, ausência de gravações e manutenção da autorização privada. Validar formato canônico, caixa, padding, correlação, headers, OpenAPI, índices, cancelamento e respostas tardias.

## Ambiente e execução

.NET SDK 10.0.401/runtime 10.0.12, Node 26.13.0/npm 11.20.0, Python 3.13.15, Liquibase 4.33.0 e PostgreSQL 18.6. Imagens fixadas pelos Dockerfiles/Compose existentes. Banco, credenciais, portas e volumes exclusivos do laboratório. Nunca usar o volume persistente. Migrador separado do runtime DML; 47 changesets originais. Dados estritamente sintéticos.

Unitários xUnit/Vitest nos projetos de aplicação. Suítes manuais em scrapbook-tests; Chromium desktop e WebKit/iPhone emulado, 320/390/768/1440px e texto 200%. Meta de carga: 1000 eventos, dez clientes por 60s, zero erros e p95 até 500ms. Emulação/carga local não certificam produção, aparelhos físicos ou acessibilidade integral.

## Evidências e falhas

Preservar resultados originais e retestes. Atribuir revisão e comando a cada resultado. Segredos, manifests com URLs de eventos, bibliotecas e logs brutos não são versionados. Publicar somente agregados e capturas revisadas; registrar testes não executados. Sem merge ou deploy neste ciclo.
