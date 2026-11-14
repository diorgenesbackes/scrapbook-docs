# Relatório: PP-STORY-005

[História](https://trello.com/c/QhI07Cia) · [Contrato](FDD-Recuperacao-da-Pagina-Publica.md) · [Execução](../../testing/2026-11-13_19-56-15/2-execucao.md)

## Resultado e base

Erros de consulta pública/compartilhamento implementados em feature/PP-STORY-005: rede, timeout, limitação, indisponibilidade, não encontrado e inesperado têm mensagens próprias, sem detail técnico ou HTML. Retry mantém rota/foco, não sobrepõe consultas e respeita Retry-After. Correlação aparece somente como UUID selecionável. Recuperação consulta a janela atual do servidor.

Antes de codificar, a stack foi parada preservando volumes. A [auditoria de todas as branches locais/remotas](../../testing/2026-11-13_19-56-15/auditoria-branches-antes.json) encontrou somente PP-STORY-003/004 pendentes em cinco projetos; Liquibase não tinha pendência. Ambas foram [integradas e enviadas às mains](../../testing/2026-11-13_19-56-15/integracao-PP-STORY-003-004.json), nessa ordem, sem conflitos e com árvore final igual à PP-STORY-004 validada. Foi executado [pull main nos seis projetos após os pushes](../../testing/2026-11-13_19-56-15/base-apos-pull-main.json), antes de criar/codificar a nova feature. Nenhuma branch anterior ficou sem merge.

Situação técnica: 33 subtarefas prontas para To Deploy. A nova história não foi integrada em main nem publicada. PP-003-034 de aparelhos/papel e PP-002-029 de leitor de tela real permanecem incompletas nas histórias anteriores, mesmo após seus merges autorizados.

## Validação

| Verificação | Resultado |
| --- | --- |
| Backend | Build sem avisos/erros; 144 unitários aprovados. Novos casos verificam Problem Details, correlação, log nominal e abandono sem falha de negócio. |
| Frontend | Lint/build e 86 unitários aprovados; sinal composto, timeout, cooldown, foco, duplicação e limite de refresh vencido cobertos. |
| HTTP real | 13 verificações nomeadas e casos adicionais de correlação; 404, 500 por ACL, 503 por conexão e lock/timeout, com recuperação e hashes preservados após leituras. |
| Recuperação no browser | 12 cenários aprovados em 22,1s; Chromium/WebKit, sem skips/retry automático. |
| Regressões | Dez janela, dez QR, 11 apresentação/performance, seis acesso público e 14 gestão/health. Total com recuperação: 63 cenários distintos. |
| Perfil 4G | Apresentação/capa/QR/janela prontos em 730/734/722/723/721ms; máximo 734ms, limite 3000ms. |
| Carga de leitura pública | 1004 eventos, dez clientes/60s: 269467 consultas, zero erros, p95 3,343ms, máximo 26,761ms. |

O lock real resultou em 503 sanitizado em 4,018s, dentro do orçamento de 5s; timeout local de 6s validado separadamente com resposta atrasada e cancelada. READ_TIMEOUT_MS não altera o orçamento do servidor nem o prazo de envio. Revogação/restauração de SELECT e NOLOGIN/LOGIN ocorreram somente no runtime/banco aleatórios, com finally e cleanup. Nenhuma permissão/role persistente foi alterada.

Chromium 158.0.8127.18/WebKit 27.2 emulados. Perfil 4G customizado: 10/2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cinco contextos novos/cache desligado. Gestão foi executada em dois laboratórios e contada uma vez. Não extrapolar para AWS, upload, aparelho físico ou rede comercial.

## Critérios de aceite

- AC01: links guest inválidos não mostram dados nem redirecionam para login. API alcançada devolve 404 padronizado; Nginx local serve página estática guest em 400 de URI inválida, sem refletir o caminho.
- AC02: retry permanece no mesmo evento, com uma consulta ativa; sinal de navegação cancela e resposta tardia não restaura outro contexto.
- AC03: loading/rede/timeout/429/indisponibilidade têm texto/ações distintos; correlação somente UUID, campo readonly selecionável e anúncio semântico.
- AC04: capa e QR falham isoladamente, preservando nome/link/cópia e geração novamente. PNG/clipboard/print mantêm os fallbacks anteriores.
- AC05: respostas e logs da API amostrados não contêm código, nome, SQL, senha ou mensagem de exceção. Erro de provider inesperado real é sanitizado; nomes de rota mantêm cardinalidade limitada.

## Entrega por subtarefa

[Transições](../../testing/2026-11-13_19-56-15/transicoes.json) usam datas UTC/actionId da atividade nativa do Trello. Não representam horas ativas exclusivas. O quadro é a situação corrente.

| ID | Entrega/evidência |
| --- | --- |
| PP-005-001 | Malformado é recusado antes do repositório; canônico ausente consulta o índice. Ambos públicos 404 RESOURCE_NOT_FOUND, sem enumerar outro evento. |
| PP-005-002 | Catálogo ReadFailure: rede/timeout/limitado/indisponível/inesperado; contratos de QR/PNG/print preservados. |
| PP-005-003 | Mensagens fixas em português, ação GET real, orientação de conexão/espera e retorno contextual. |
| PP-005-004 | Sem retry de escrita/erro automático; lease vencido permite só uma reconsulta extra em 250ms, impedindo loop infinito. |
| PP-005-005 | Timeout local 6s, API 5s/provider 4s, cancelamento vinculado à navegação. |
| PP-005-006 | UUID canônico selecionável, sem envio de mensagem ou URL/código do evento no diagnóstico da UI. |
| PP-005-007 | PublicEventController trata GET vazio/catch-all e preserva precedência da rota válida; Problem Details comum. |
| PP-005-008 | Mapper existente confirmado com conexão bloqueada e timeout reais, retornando 503 sem detalhes do provider. |
| PP-005-009 | Middleware cobre inesperado em unidade e 500 real por ACL revogada no banco descartável. |
| PP-005-010 | Correlação válida preservada; inválida/múltipla substituída; header/body coerentes em erros alcançados pela API. |
| PP-005-011 | Token até EF preservado; aborto de navegação propagado ao sinal composto, resposta tardia ignorada e client_aborted sem Problem Details de negócio. |
| PP-005-012 | Logger usa operação/duração/status/classe, sem mensagem de exceção ou corpo; amostra real aprovada. |
| PP-005-013 | Rotas nominais estáticas, inclusive inválidas; código do evento não entra em tags/labels. |
| PP-005-014 | Shell guest de inválido/ausente e fallback estático seguro para 400 antes de carregar a SPA. |
| PP-005-015 | Falha de rede orienta conferir conexão, com retry guardado. |
| PP-005-016 | Indisponibilidade/inesperado exibem mensagem própria e referência somente se UUID. |
| PP-005-017 | Proxy/WAF HTML não é renderizado como diagnóstico; status suficiente para classificar. |
| PP-005-018 | readJson cancela em 6s mesmo se um transport mock ignora sinal; aborto de navegação fica neutro. |
| PP-005-019 | Guarda síncrona no hook/loader e aria-disabled no botão coalescem cliques; 429 bloqueia foco/online durante cooldown. |
| PP-005-020 | Código/ID da rota permanecem no retry; validação da resposta e cleanup impedem mistura entre eventos. |
| PP-005-021 | Fallbacks de capa/clipboard/QR/PNG/print reutilizados; QR com aria-live, link textual e geração manual. |
| PP-005-022 | Controle permanece no DOM; foco manual preservado/recuperado sem retirar foco de outro campo. |
| PP-005-023 | Banco recuperado e data alterada pela gestão resultam em closed novo, sem restaurar open anterior. |
| PP-005-024 | Sete novos casos xUnit de erro/log/correlação/cancelamento, total 144. |
| PP-005-025 | HTTP vazio/malformado/1024 caracteres/encoding/ausente. %GG no Nginx: 400 de transporte; API direta alcançada: 404. |
| PP-005-026 | ACL, NOLOGIN e lock exclusivos com restauração; recuperações retornam projeção pública válida. |
| PP-005-027 | Status, application/problem+json, no-store e UUID/body verificados no pipeline; edge anterior tem limite explícito. |
| PP-005-028 | Unitários/browser: offline, resposta lenta, 404/429/5xx e corpo HTML/JSON inválido. |
| PP-005-029 | Retry repetido, unmount/navegação e resposta após 7s ignorada; nenhuma consulta duplicada. |
| PP-005-030 | Recuperação real de conexão sem reload ou login, preservando nome/QR e evento da rota. |
| PP-005-031 | Capa bloqueada e canvas QR indisponível preservam alternativas; geração retomada por Enter. |
| PP-005-032 | Mensagem visível/semântica, referência legível, foco e ativação por teclado em Chromium/WebKit. Não é execução com leitor de tela real. |
| PP-005-033 | FDD, operação, plano/execução, falhas/retestes, revisões, capturas e auditoria dos merges/pulls consolidados. |

## Falhas, limites e recuperação

[Falhas e ajuste de revisão preservados](../../testing/2026-11-13_19-56-15/falhas-e-correcoes.json): sintaxe TS incompatível com erasableSyntaxOnly; dois testes antigos de sinal/texto; suposição de JSON em 400 do proxy; primeira rodada de browser 7/12; segundo reteste 11/12 pelo Tab inicial no perfil WebKit. Option+Tab segue a convenção já usada no projeto. O diagnóstico final não atribui essa falha inicial a perda de foco após retry. Uma revisão posterior identificou que ocultar uma aba com evento conhecido podia substituir visualmente o erro 429 por estado genérico, embora a consulta continuasse bloqueada. A UI agora preserva mensagem/cooldown; um novo unitário cobre ocultação, retorno e retry. Execução de recuperação final 12/12 aprovada sem retry automático; timestamps individuais não foram inventados.

As regressões completas e a medição 4G usaram a UI 85097c2, registrada em [revisões iniciais](../../testing/2026-11-13_19-56-15/revisoes-regressoes-iniciais.json). O ajuste final b80ff23 altera somente a preservação do erro no estado stale: lint/build, 86 unitários e as 12 jornadas de recuperação foram retestados. Não se repetiu a carga da API, cujo código permaneceu 49a60a9, nem a medição da apresentação saudável; não afirmar equivalência binária entre os bundles. Os itens afetados voltaram a Doing/Test no Trello antes da conclusão.

Capturas selecionadas foram inspecionadas. Teclado/árvore semântica não comprovam leitor real ou acessibilidade integral. Nginx local em 400 não comprova configuração de Amplify/WAF externos. Correlacionar erro do edge depende dos headers realmente disponíveis; não gerar referência fictícia. Logs/snapshots são amostras locais, sem alegar SLA.

[Revisões de entrega](../../testing/2026-11-13_19-56-15/revisoes.json). Sem pacote novo, DDL/migration, autenticação ou AWS. Edição/restauração de fixture muda evento/auditoria intencionalmente; demais tabelas preservadas. Aplicação persistente parada e volumes preservados, laboratórios removidos. Publicar API/fallback de hosting antes da nova UI; recuperação reverte UI e depois API, sem alteração de banco. Sem merge/deploy da PP-STORY-005.
