# Execução: PP-STORY-004

[Plano](1-plano.md) · [Relatório por subtarefa](../../fdd/FDD-Disponibilidade-dos-Envios/relatorio.md)

## Base e revisões

Stack persistente parada com volumes preservados. feature/PP-STORY-004 criada de main em service/UI/tests/docs/infrastructure, incluindo PP-STORY-003 somente na feature para preservar link/QR. [Base, dependência e merge interno](dependencias-e-base.json); promover PP-STORY-003 primeiro. Main e Liquibase não foram alterados nesta rodada. Sem merge/deploy.

[Revisões finais](revisoes.json) identificam os quatro componentes de código/operação; documentação consolidada em commit posterior da mesma feature.

## Execuções

- `dotnet build ScrapBook.sln --no-restore --disable-build-servers` e `dotnet test ScrapBook.sln --no-build --no-restore`: build sem avisos/erros, 137 aprovados. Os testes verificam bordas em ticks, calendário, galeria independente e uma leitura do TimeProvider por resposta.
- `npm run check`: lint/build e 71 testes aprovados na versão final, com cancelamento/timers/foco/online/aba oculta, referência antiga, transporte e estado não encontrado preservado.
- `dotnet build probes/ScrapBook.Probes.csproj --no-restore --disable-build-servers`: aprovado, sem pacote/referência nova. Host temporal usa Kestrel e componentes reais, mas composição mínima própria, não o Program completo de produção.
- `python3 scripts/stack_local.py build`: imagens finais construídas; não inicia a stack persistente.
- `python3 submission_window.py`: [resultado original](submission-window-report.json), execução final 19:54:45–19:55:56 UTC. 36 casos HTTP aprovados com relógio por arquivo privado; dez browser aprovados em 32,6s, sem skips/retry. [Resumo](temporal-browser-summary.json). Casos de galeria/edição usam a API real. Parada/retorno do container service testada uma vez por browser, sem extensão dos marcos. TimeProvider de produção permanece System, sem endpoint de alteração de hora.
- `python3 event_sharing.py`: [regressão de QR](regressao-compartilhamento.json), 19:56:45–19:58:12 UTC. Dez browser aprovados em 5,6s; autorização de dois responsáveis/outsider, origem configurada/ausente, Host adulterado, OpenAPI, leitura sem escrita e logs aprovados. PNG/print digital decodificados por leitor independente; não comprova papel/aparelho real.
- `python3 public_presentation.py`: [regressão de apresentação](regressao-apresentacao.json), 19:58:24–20:00:49 UTC. 11 browser de apresentação/performance aprovados em 7,8s; 14 jornadas de gestão/health aprovadas. Metadados alterados/restaurados, projeção pública, autorização, snapshots e logs aprovados.
- Mesma execução de apresentação: 1004 eventos, dez clientes, 60s, 264262 consultas, zero erros, p95 3,437ms e máximo 25,675ms. Metas locais atendidas; mede leitura pública, não uploads ou AWS.
- `python3 public_event.py --functional-only`: [regressão de acesso direto/reload/nova aba](regressao-acesso-publico.json), reteste final 22:28:48–22:30:05 UTC. Seis testes públicos aprovados em 2,8s e 14 de gestão em 7,2s. Carga não repetida. A execução interrompida pelo Docker não conta como aprovada; somente o reteste final é evidência.

Total: 51 cenários distintos (10 temporais + 10 compartilhamento + 11 apresentação/performance + seis acesso público + 14 gestão). A gestão foi executada em dois laboratórios e contada uma vez.

## Performance e limites

[Perfil/medições](performance-4g.json): Chromium 158.0.8127.18, 390×844, download 10Mbps/upload 2Mbps decimais, latência mínima de headers 80ms, CPU slowdown 4×, cache desligado/contexto novo por amostra. Prontidão inclui nome/mensagem/capa/QR e disponibilidade; 747/734/707/731/709ms, máximo 747ms (meta 3000ms). Não extrapolar para rede comercial ou aparelho real. WebKit 27.2 móvel emulado.

O browser temporal fixa Date em 2099 e fusos Tokyo/Los Angeles, mantendo timers monotônicos. Só o GET público do fixture é encaminhado ao host Kestrel de teste; a parada real usa exclusivamente Nginx/API Compose. Matriz temporal testa ±1ms em HTTP; precisão em ticks é unitária. Anúncio semântico/foco foi validado, sem leitor de tela real.

## Falhas e retestes

[Registro consolidado](falhas-e-correcoes.json): compilação do novo teste com nome de propriedade incorreto; lint exigindo output em lugar de div role=status; health duplicado do host de testes, corrigido após duas execuções que não alcançaram o browser; Docker Desktop indisponível durante preparação/snapshot da regressão de acesso. Sem timestamp inventado por ocorrência.

Health do probe foi verificado isoladamente como 200 após a correção. Matriz e browser passaram e foram retestados após ajuste de preservação do estado não encontrado e sincronização explícita da borda de encerramento. Falhas de setup não foram ocultadas por retry automático. A indisponibilidade do Docker impediu o finally de remover seu laboratório; [limpeza dos três containers, dois volumes e rede exclusivos](limpeza-laboratorio-interrompido.json) foi executada após reabrir o Docker, preservando scrapbook-local.

## Evidências e rastreio

[Antes/desktop](submission-window-before-chromium-desktop.png) · [Antes/WebKit](submission-window-before-webkit-mobile.png) · [Encerrado/desktop](submission-window-closed-chromium-desktop.png) · [Encerrado/WebKit](submission-window-closed-webkit-mobile.png).

[Transições por tarefa](transicoes.json) vêm da atividade nativa do cartão Trello, com data UTC/actionId. Todo → Doing → Test → To Deploy, sem inferir horas ativas. Situação de entrega: 35 subtarefas prontas; PP-003-034 de aparelhos/papel e PP-002-029 de leitor real continuam separadamente pendentes.

Domain/Application sem dependência de ASP.NET/EF; projeção SQL mínima e no-store. Snapshots preservados após leituras; edição/restauração de fixtures altera somente evento/auditoria intencionalmente. Sem DDL/migration/pacote novo. Stack persistente parada, volumes preservados; laboratórios exclusivos limpos. Sem merge em main, publicação ou AWS.
