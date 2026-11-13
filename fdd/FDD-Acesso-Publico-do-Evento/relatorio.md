# Relatório: PP-STORY-001

[Contrato](FDD-Acesso-Publico-do-Evento.md) · [História](https://trello.com/c/LkDStr6e) · [Execução completa](../../testing/2026-11-13_14-07-57/2-execucao.md)

## Resultado

Implementada consulta pública somente leitura por código, com shell guest separado da gestão. GET /api/v1/public/events/{code} retorna exclusivamente code; /e/:code abre diretamente sem login. Código/PK inválido e ausência produzem 404. Não há gravação em banco, nem resolução de identidade no caminho público. Privacidade de dados e proteção das rotas privadas verificadas localmente.

Entrega técnica na feature/PP-STORY-001 em service, UI, tests, infrastructure e docs. Sem merge para main, sem deploy e sem mudanças no Liquibase. Nome/capa/mensagem, compartilhamento/QR e envio continuam nas histórias posteriores.

## Validação

| Verificação | Resultado |
| --- | --- |
| Restore locked/build .NET | Passou, zero avisos/erros no build. |
| xUnit backend | 99 passaram, zero ignorados. |
| UI lint/build/Vitest | Passou; 48 testes. |
| HTTP / PostgreSQL / OpenAPI | Seis grupos aprovados; índice existente utilizado, 47 migrations preservadas. |
| Playwright público | Seis testes finais aprovados em Chromium 158.0.8127.18 / WebKit 27.2. |
| Regressão de gestão/health | 14 testes aprovados nos mesmos projetos. |
| Carga pública | 1000 eventos, dez concorrentes, 60s; 279801 requests, zero erros, p95 3,176 ms, máximo 50,529 ms. |
| Privacidade | Payload mínimo; tabelas de aplicação preservadas após leituras/navegador/carga; amostra de logs sem códigos ou textos privados. |

A carga ocorreu no ciclo 1 sobre a mesma API final. O ciclo 3 revalidou o foco do shell após adicionar tabindex ao destino do skip link; não repetiu carga, explicitamente marcada como não executada nesse ciclo. Falha intermediária: o teste de Shift+Tab não incluía links no modo padrão de teclado do WebKit. Uma página HTML sintética confirmou Option+Tab como navegação por links, e o cenário foi corrigido/reexecutado com sucesso. Os resultados originais foram preservados.

## Matriz de aceite

| Critério | Evidência |
| --- | --- |
| AC01 acesso anônimo ao evento correto | HTTP em Production com identidade desabilitada; Playwright sem cookie/Authorization e com resposta vinculada ao código da rota. |
| AC02 código existente, opaco e case-sensitive | Unitários, testes HTTP de caixa/PK/padding e snapshot do banco. |
| AC03 acesso direto, reload e nova aba | Seis cenários públicos no Nginx do Compose; configuração Vite preservada. Rewrite real no Amplify depende da implantação futura. |
| AC04 ausência de gravações | Digest de todas as tabelas dos schemas identity/events/records/notifications antes e após leituras; somente fixtures de teste fazem gravações explícitas. |
| AC05 proteção da gestão | Ator alheio recebe 404; visitante sem identidade recebe 401 em GET/PUT privados; código público não substitui PK/credencial. |

## Entrega por subtarefa

Estado e transições reais ficam no Trello e em [transições](../../testing/2026-11-13_14-07-57/transicoes.json). Horários representam observação após operações do conector, não medição de horas ativas exclusivas. Nenhum item foi movido para Done ou publicado.

| ID | Entrega e evidência |
| --- | --- |
| PP-001-001 | Contrato SPA /e/:code, FDD seção 4. |
| PP-001-002 | GET público e GetPublicEvent no OpenAPI, validado em HTTP. |
| PP-001-003 | PublicEventCodePolicy e unitários de formato/caixa/padding. |
| PP-001-004 | PublicEventResponse exclusivo com code; assert exato do payload. |
| PP-001-005 | Middleware, correlação, no-store e orçamento de 5s documentados. |
| PP-001-006 | Repositório de leitura e snapshots idênticos nas tabelas de aplicação. |
| PP-001-007 | PublicEventModel puro em Domain; teste de dependências aprovado. |
| PP-001-008 | PublicEventResponse puro e separado do contrato administrativo. |
| PP-001-009 | IPublicEventRepository / IPublicEventService sem ator ou escrita. |
| PP-001-010 | Projeção AsNoTracking parametrizada em PublicEventRepository. |
| PP-001-011 | EXPLAIN comprovou uso do índice existente de code; sem migration. |
| PP-001-012 | PublicEventService usa porta somente leitura; unitários e HTTP. |
| PP-001-013 | PublicEventController real em Api/Controllers, com AllowAnonymous. |
| PP-001-014 | Validação de código, token e correlação verificados em unitários/HTTP. |
| PP-001-015 | DI registrada; acesso privado continuou exigindo ator/vínculo. |
| PP-001-016 | App separa rotas/layout guest da gestão; navegador anônimo aprovado. |
| PP-001-017 | Cliente tipado omite cookies e não reutiliza contrato de gestão. |
| PP-001-018 | AbortController verificado ao trocar/sair; sem efeito de resposta antiga. |
| PP-001-019 | Unitários cobrem sucesso e falha tardios depois de trocar evento. |
| PP-001-020 | Shell guest com foco inicial, status, skip link e composição responsiva. |
| PP-001-021 | Fallback Nginx existente confirmado; assets/API não viram HTML da SPA. |
| PP-001-022 | README de infraestrutura com URLs locais e rewrite futuro do Amplify. |
| PP-001-023 | Unitários xUnit de validade, comprimento, caixa e bits canônicos. |
| PP-001-024 | Unitários xUnit de sucesso, ausência, cancelamento e zero I/O inválido. |
| PP-001-025 | Vitest cobre abertura, cancelamento, troca de rota e callbacks antigos. |
| PP-001-026 | HTTP funciona em Production sem opt-in local, conta, cookie ou token. |
| PP-001-027 | PK/código não liberam gestão; anônimo 401 e ator alheio 404. |
| PP-001-028 | Snapshots em todos os schemas de aplicação idênticos após leituras. |
| PP-001-029 | Seis cenários públicos e 14 de regressão em Chromium/WebKit; foco e nova aba. |
| PP-001-030 | Carga com 1000 eventos/10 concorrentes/60s: 279801 requests, zero erros, p95 3,176 ms. |
| PP-001-031 | FDD, READMEs, relatório, hashes e comandos reais documentados. |
| PP-001-032 | Transições observadas em UTC, histórico de falha/reteste e evidências vinculadas. |

## Limitações e recuperação

Validação local em containers descartáveis; sem AWS, aparelhos físicos, leitor de tela real, auditoria WCAG completa, ensaio de disponibilidade mensal ou aceite visual integral. O contrato mínimo é deliberado e não confirma disponibilidade de envio. A stack persistente e os volumes do usuário permaneceram parados/preservados; laboratórios foram removidos.

Ordem futura: API antes da UI. Reverter UI e depois API para as revisões anteriores, sem rollback de banco pois não houve DDL/DML de funcionalidade. Todas as dependências permanecem nas versões existentes; manifests/lockfiles preservados e bibliotecas/artefatos privados ignorados.
