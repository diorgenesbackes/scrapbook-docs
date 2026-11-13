# Execução: PP-STORY-001

[Plano](1-plano.md) · [Contrato](../../fdd/FDD-Acesso-Publico-do-Evento/FDD-Acesso-Publico-do-Evento.md) · [Relatório por subtarefa](../../fdd/FDD-Acesso-Publico-do-Evento/relatorio.md)

## Revisões avaliadas

- scrapbook-service: `9ee00415a53b8c1841a5c627c64d5de05c3b8748`.
- scrapbook-ui: `6194750e67bc16ab28fbe28546536c1200d6959e`.
- scrapbook-tests: `16b9686330974b9806e10960b15fc2f574dd09f5`.
- scrapbook-infrastructure: `bf3dba123fe6d199a7c4292361e8a39e0d6e5e30`.

Documentação e transições são consolidadas em commit posterior da mesma feature. Nenhum merge ou deploy foi executado.

## Comandos e resultados

- service: `dotnet restore ScrapBook.sln --locked-mode --disable-build-servers`, `dotnet build ScrapBook.sln --no-restore --disable-build-servers` e `dotnet test ScrapBook.sln --no-build --no-restore`: restore e build aprovados, zero avisos/erros, 99 testes passando. Execução via autorização do sandbox; tentativa inicial restrita de restore encerrou sem saída útil.
- UI: `npm run check`: lint/build aprovados e 48 testes. A primeira execução encontrou setState síncrono no effect e uso de role=status em vez de output; corrigidos com estado derivado da rota/tentativa e output semântico. Checks finais aprovados.
- infrastructure: `python3 scripts/stack_local.py build`; depois `docker compose build ui` para o ajuste do skip link. Imagens locais construídas; nenhuma stack persistente iniciada. Uma revisão automática da reconstrução falhou internamente com Bad Request sem executar a ação; repetição idêntica foi aprovada e executada.
- tests, ciclo 1: `python3 public_event.py`: contratos/DB/OpenAPI, seis testes públicos e carga passaram. [Resultado original](public-event-cycle-1.json): 279801 consultas/60s, dez clientes, zero erros, p95 3,176ms, máximo 50,529ms. API idêntica à final; UI anterior apenas ao tabindex adicional no destino do skip link.
- tests, ciclo 2: `python3 public_event.py --functional-only`: contratos passaram; cinco testes públicos passaram e um falhou ao esperar Shift+Tab no WebKit. [Resultado preservado](public-event-cycle-2.json). O runner encerrou e removeu seus recursos; não houve carga ou regressão de gestão nesse ciclo.
- diagnóstico local: HTML sintético em WebKit desktop/móvel confirmou Shift+Tab focando body e Alt+Shift+Tab focando o link. Cenário ajustado para Option/Alt+Tab, sem alterar a regra funcional do produto.
- tests, ciclo 3: `python3 public_event.py --functional-only`: contratos, seis testes públicos (2,7s) e 14 de regressão de gestão/health (7,4s) passaram. [Resultado final](public-event-cycle-3.json). Carga explicitamente não repetida: mudança somente no foco da UI e no teste. Chromium 158.0.8127.18 e WebKit 27.2; sem skips e sem retry automático.

## Dados, privacidade e limpeza

Consulta pública em Production, sem identidade local e sem credencial do convidado. Payload exato {code}; código inválido/PK/caixa/padding testados. Isolamento privado preservado. EXPLAIN confirmou uso de índice de code existente. Digests de todas as tabelas de aplicação comparados antes/depois de leituras, navegador e carga. Fixtures adicionais da regressão de gestão são gravações explícitas posteriores à asserção de somente leitura. Os 47 changesets foram preservados.

Logs amostrados após consultas/carga sem códigos públicos utilizáveis nem marcadores de nome/mensagem; scopes automáticos do console desativados para impedir RequestPath. Não equivale à auditoria integral de todos os erros/serviços externos de PP-STORY-006.

Cleanup removeu somente os containers/volumes dos projetos aleatórios do laboratório. Banco e serviços persistentes do usuário não foram iniciados nem alterados.

## Evidências visuais e limites

[Desktop](public-event-chromium-desktop.png) · [WebKit móvel emulado](public-event-webkit-mobile.png) · [Revisões](revisoes.json) · [Transições observadas em UTC](transicoes.json).

Inspeção visual local do shell mínimo efetuada. Nome/capa/mensagem e QR real não integram esta história. Não executados: deploy/AWS, aparelhos físicos, leitor de tela real, homologação, recuperação regional, mensageria/upload ou auditoria WCAG integral. Aceite humano e publicação permanecem separados da entrega técnica.
