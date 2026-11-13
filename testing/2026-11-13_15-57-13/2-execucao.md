# Execução: PP-STORY-003

[Plano](1-plano.md) · [Relatório por tarefa](../../fdd/FDD-Compartilhamento-do-Evento/relatorio.md)

## Base e revisões

Stack persistente parada antes dos merges. [PP-STORY-002 integrada nas mains de service/UI/tests/docs](integracao-PP-STORY-002.json) por solicitação do responsável, sem conflitos e com árvores iguais às features validadas. PP-002-029 continua pendente; merge não significa conclusão desse teste. feature/PP-STORY-003 criada da main em cinco componentes; Liquibase permanece inalterado.

- scrapbook-service: `bda12c53d7c43d3ec9f8c76125fde9bbd52fc9cb`.
- scrapbook-ui: `d15eed7dba3999af0cd6d1c0b1c5a7f5c1efda6a`.
- scrapbook-tests: `34695be8a430ef70562a2b4b5e783057b62385e5`.
- scrapbook-infrastructure: `0cb1462a7f0d2b4233475b60cf57381f2a8aa081`.

[Revisões em JSON](revisoes.json). Documentação/transições consolidadas em commit posterior da mesma feature; sem merge da PP-STORY-003.

## Comandos e resultados

- `dotnet build ScrapBook.sln --no-restore --disable-build-servers`: zero avisos/erros. `dotnet test ScrapBook.sln --no-build --no-restore`: 120 aprovados.
- `npm run check`: lint/build e 62 testes aprovados após corrigir spy do teste de download. Ajuste final de CSS de impressão também foi construído no Docker e validado no navegador.
- `python3 event_sharing.py`: [resultado original](event-sharing-report.json), execução final 18:57:13–18:58:36 UTC. API privada mínima para dois responsáveis, terceiro negado, Host/Forwarded ignorados, origem HTTPS sintética/ausente, OpenAPI, snapshots somente leitura, privado sem identidade e logs amostrados aprovados. Dez testes Playwright aprovados em 5,6s. [Resumo](sharing-browser-summary.json). Sem skips/retry.
- `python3 public_presentation.py`: [regressão final](regressao-apresentacao.json), 19:00:59–19:03:25 UTC. 11 testes de apresentação/performance aprovados em 8,1s; 14 de gestão/health em 8,3s. Alteração/restauração de mensagem, OpenAPI, snapshots e logs aprovados.
- Mesma execução: 1004 eventos/dez clientes/60s, 265946 consultas públicas, zero erros, p95 3,43ms e máximo 37,018ms. Metas locais atendidas; sem extrapolação para AWS/uploads ou rota privada de compartilhamento.
- `python3 public_event.py --functional-only`: [regressão do acesso original](regressao-acesso-publico.json), 19:04:48–19:06:09 UTC. Seis testes públicos aprovados em 3,2s, 14 de gestão em 8,5s; carga não repetida. Total de 41 cenários distintos (10+11+6+14), contando gestão uma vez.

Chromium 158.0.8127.18 e WebKit 27.2. Emulação móvel e fusos distintos da gestão preservados. Nenhuma execução física ou leitor de tela real declarado.

## QR, print e performance

jsQR 1.4.0/pngjs 7.0.0, independentes de qrcode 1.5.4, decodificaram bitmap gerado, screenshot da prévia, PNG baixado e screenshot de print para a mesma URL. Margem digital e nome de arquivo seguros conferidos. Contexto anônimo novo abriu o destino e exibiu o evento correto. Origem externa sintética não foi acessada.

Mídia print usa portal, nome/link/QR, fundo html/body branco e exclui controles. Chamada window.print foi simulada para não abrir diálogo do sistema; composição foi verificada por mídia print do navegador. Isso não comprova impressora, diálogo real ou leitura do papel. **PP-003-034 permanece Test/incompleta.**

[Perfil 4G](performance-4g.json): 10/2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cinco contextos novos/cache desabilitado. Prontidão inclui nome/mensagem/capa e QR visíveis; 741/739/742/769/736ms, máximo 769ms (meta 3000ms). Medição local não caracteriza rede comercial.

## Falhas e retestes

[Registro](falhas-e-correcoes.json) preserva quatro ocorrências com horário observado:

1. Primeiro unitário UI: 61/62, spy de anchor perdido por restauração global de mocks. Spy recriado em beforeEach; 62/62 no reteste.
2. Fixture do segundo responsável sem joined_at: rejeitada antes do navegador. Campo preenchido explicitamente conforme schema.
3. Fixture de vínculo sem convite aceito: constraint diferida existente impediu transação. Usada transação sintética válida de convite aceito/vínculo, como na suíte existente, sem mudança de produto/schema.
4. Proxy devolveu resposta transitória não JSON após recriar API. Helper agora aguarda GET /health através do Nginx com limite de 30s; sem retry de escrita ou asserção funcional. Regressões finais aprovadas.

Inspeção visual detectou fundo externo bege na primeira composição de print; CSS branco e asserção adicionados. As seis capturas finais documentam S15, PNG e print em ambos os browsers.

## Evidências e situação

[S15 desktop](event-sharing-chromium-desktop.png) · [S15 móvel emulado](event-sharing-webkit-mobile.png) · [PNG desktop](event-sharing-material-chromium-desktop.png) · [PNG móvel](event-sharing-material-webkit-mobile.png) · [Print desktop](event-sharing-print-chromium-desktop.png) · [Print móvel](event-sharing-print-webkit-mobile.png).

[Transições](transicoes.json): 143 observações UTC após operações do conector. Todos passaram por Todo/Doing/Test; 35 chegaram a To Deploy/concluídos. PP-003-034 ficou Test/incompleta, história Test. Não converter intervalos em horas ativas ou atribuir timestamp nativo do Trello. PP-002-029 de leitor real continua separadamente pendente.

Domain/Application permanecem sem referências de ASP.NET/EF; EF em Infrastructure, opções/HTTP em API. Sem DDL novo, alterações no volume persistente ou libs versionadas. Snapshots das tabelas preservados após leituras; fixtures/regressão privada fazem gravações intencionais separadas. Logs amostrados redigidos. Laboratórios exclusivos removidos no finally; stack persistente parada com volumes preservados. Sem AWS, merge da nova feature ou deploy.
