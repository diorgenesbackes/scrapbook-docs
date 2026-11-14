# Execução: PP-STORY-005

[Plano](1-plano.md) · [Relatório por tarefa](../../fdd/FDD-Recuperacao-da-Pagina-Publica/relatorio.md)

## Integração e base antes da codificação

[Auditoria de branches locais/remotas](auditoria-branches-antes.json): somente PP-STORY-003/004 pendentes em service/UI/tests/docs/infrastructure, sem workflow de publicação nos trees auditados. Liquibase sem pendência. [Merges](integracao-PP-STORY-003-004.json) no padrão --no-ff, 003 antes de 004, sem conflitos, árvore final igual à PP-STORY-004 já validada. Push nas cinco mains concluído.

[Pull main nos seis projetos após os pushes](base-apos-pull-main.json), com heads iguais ao remoto, antes de criar/codificar feature/PP-STORY-005 diretamente da main. Todas as branches anteriores eram ancestrais. Pendências PP-003-034 e PP-002-029 mantidas, sem tratar merge como deploy ou QA concluído.

## Revisões e comandos

[Revisões finais](revisoes.json): service 49a60a9, UI b80ff23, infrastructure db11a5a e tests 153640f. Documentação em commit posterior da mesma feature.

- `dotnet build ScrapBook.sln --no-restore --disable-build-servers` e `dotnet test ScrapBook.sln --no-build --no-restore`: zero avisos/erros, 144 unitários aprovados.
- `npm run check`: lint/build e 86 testes aprovados na versão final. Alterações posteriores de interface/fallback foram construídas em Docker e retestadas em browser.
- `python3 scripts/stack_local.py build`: imagens finais, sem subir a stack persistente.
- `python3 public_recovery.py`: [resultado original](public-recovery-report.json), 23:50:36–23:51:41 UTC. HTTP: 13 verificações nomeadas, correlação adicional, ACL/NOLOGIN/lock e recuperação. Timeout de lock 4,018s. Doze testes browser aprovados em 22,1s, sem skips/retry automático. [Resumo](recovery-browser-summary.json).
- `dotnet build probes/ScrapBook.Probes.csproj --no-restore --disable-build-servers`: aprovado, atualizando o host exclusivo temporal para a API corrente.
- `python3 submission_window.py`: [regressão temporal](regressao-janela.json), 36 casos HTTP e dez browser aprovados, incluindo parada/retorno de service sem extensão de prazo. Host de testes por arquivo privado, sem controle de relógio na API de produção.
- `python3 event_sharing.py`: [regressão de compartilhamento](regressao-compartilhamento.json), dez browser aprovados; dois responsáveis/outsider, origem/Host, QR/PNG/print digital e leitura sem escrita preservados.
- `python3 public_presentation.py`: [regressão de apresentação](regressao-apresentacao.json), 11 browser de apresentação/performance e 14 de gestão/health aprovados. [Medições 4G](performance-4g.json), com apresentação/capa/QR/janela prontos em até 734ms.
- Mesma execução: 1004 eventos, dez clientes/60s, 269467 consultas públicas, zero erros, p95 3,343ms e máximo 26,761ms. Não mede uploads/AWS.
- `python3 public_event.py --functional-only`: [regressão de acesso direto/reload/nova aba](regressao-acesso-publico.json), 23:33:53–23:35:08 UTC. Seis browser públicos aprovados em 2,2s e 14 de gestão em 7,2s. Carga não repetida.

Total: 63 cenários distintos (12 recuperação + 10 janela + 10 QR + 11 apresentação/performance + seis acesso + 14 gestão). Os 14 de gestão foram executados em dois laboratórios, contados uma vez.

## Falhas e recuperação observada

[Registro](falhas-e-correcoes.json): TS1294 no construtor do novo erro, corrigido com campo explícito; 69/71 unitários iniciais, dois exigindo sinal original/texto técnico; JSON esperado em rejeição %GG do proxy; primeira rodada browser 7/12 e reteste 11/12. Correções: fallback estático Nginx 400, timeout do matcher para aguardar os 6s e Option+Tab no WebKit. A falha WebKit era no foco inicial, antes do retry, não perda observada após retry. Versão final 12/12; nenhum resultado foi aprovado por retry automático.

Uma revisão final identificou que ocultar/retornar a aba com evento conhecido e 429 mantinha o bloqueio de consultas no hook, mas podia perder a indicação visual de espera. O erro/cooldown agora é preservado também em stale, com novo unitário e 86 aprovados. Os itens 019/028/033 voltaram a Doing/Test para o ajuste e a documentação. A UI final b80ff23 passou novamente nas 12 jornadas de recuperação.

As regressões completas/perfil 4G usaram a UI 85097c2, com [revisões iniciais](revisoes-regressoes-iniciais.json). O ajuste final atinge somente a exibição do erro em stale; não repetiu a medição saudável ou a carga da API 49a60a9 inalterada. As revisões finais não implicam bundles binariamente iguais aos iniciais.

API direta aceita o caminho %GG até seu handler e retorna 404 sanitizado; o Nginx o recusa antes da API com 400. Documento inválido recebe HTML estático em português sem scripts/URI refletida, mantendo status/no-store. API 400 JSON continua sem interceptação pelo proxy. Isso não equivale a configurar WAF/Amplify remotos.

REVOKE/GRANT SELECT provocou 500 real sem detalhes do provider; NOLOGIN/LOGIN com encerramento das conexões provocou 503; lock exclusivo confirmou timeout e recuperação. Tudo somente no papel/banco descartáveis, restaurado no finally. Hashes das tabelas de aplicação iguais após leituras. Edição/restauração da data em browser é gravação intencional de fixture/auditoria; startsAt restaurado e demais tabelas iguais.

## Perfil e evidências

Chromium 158.0.8127.18/WebKit 27.2 emulados. Perfil 4G: 10/2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cinco contextos novos/cache desabilitado. Tempos 730/734/722/723/721ms; limite 3000ms. Medição local não caracteriza rede comercial, dispositivo real, produção ou SLA.

[Erro desktop](public-recovery-service-chromium-desktop.png) · [Erro WebKit](public-recovery-service-webkit-mobile.png) · [Recuperado desktop](public-recovery-restored-chromium-desktop.png) · [Recuperado WebKit](public-recovery-restored-webkit-mobile.png). Capturas selecionadas inspecionadas visualmente.

[Transições nativas UTC](transicoes.json) identificam Todo → Doing → Test → To Deploy por tarefa/actionId, sem inferir horas ativas. Situação técnica: 33 subtarefas prontas; QA humano anterior preservado. Teclado/mensagem semântica não substituem leitor de tela real.

Domain/Application puros, sem pacote novo, DDL ou migration; mesmo histórico de 47 changesets. Logs da API amostrados sem nome/código/detalhes SQL. Stack persistente parada, volumes preservados; cleanup só dos projetos aleatórios. Nenhum merge da nova feature, autenticação, upload ou AWS.
