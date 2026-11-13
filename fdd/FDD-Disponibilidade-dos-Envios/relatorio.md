# Relatório: PP-STORY-004

[História](https://trello.com/c/lDmeo2EA) · [FDD](FDD-Disponibilidade-dos-Envios.md) · [Execução](../../testing/2026-11-13_16-55-35/2-execucao.md)

## Resultado

Disponibilidade implementada em feature/PP-STORY-004: API pública retorna not_open/open/closed, serverTime UTC, marcos civis de Brasília e prazo curto para revalidar. A política reutiliza EventPolicy.Window; PublicEventService amostra o relógio uma vez após consultar os dados mínimos. Abertura inclusiva duas horas antes; encerramento exclusivo para novos envios ao meio-dia seguinte. Entre 12h e 13h permanece closed.

A UI exibe os horários civis sem reinterpretar o fuso do aparelho. Agenda GET pela duração monotônica, desconta o transporte, revalida nos limites/em até 30s e ao voltar à aba ou recuperar conexão. Consulta, falha e resposta antiga retiram a afirmação de disponibilidade; nome/capa/link/QR conhecidos continuam visíveis. Nenhum timer autoriza envio ou avança o estado por conta própria.

A stack persistente foi parada com volumes preservados. A feature nasceu de main e incluiu PP-STORY-003 somente na própria feature, com árvores iniciais iguais à dependência. [Bases e revisões](../../testing/2026-11-13_16-55-35/dependencias-e-base.json). Promover PP-STORY-003 antes desta história. Nenhum merge em main ou deploy nesta rodada.

**Situação de entrega:** 35 subtarefas prontas para To Deploy. PP-003-034 de leitura em aparelhos/papel e PP-002-029 de leitor de tela real continuam pendentes nas histórias anteriores; não são concluídas por esta validação.

## Validação e aceite

| Verificação | Resultado |
| --- | --- |
| Backend | Build sem avisos/erros; 137 testes xUnit aprovados, incluindo arquitetura pura, bordas em ticks e amostra única do relógio. |
| Frontend | Lint/build e 71 testes Vitest aprovados, incluindo transporte, relógio civil divergente, timers, ocultação/foco/online e resposta antiga. |
| HTTP temporal | 36 casos com PostgreSQL real e TimeProvider exclusivo de testes: abertura/12h ±1ms, 12h30/13h/depois, madrugada, mês/ano/bissexto. |
| Navegador temporal | Dez cenários Chromium/WebKit aprovados; Date em 2099, fusos Tokyo/Los Angeles, bordas em página aberta, edição real e parada/retomada do container. |
| Regressões | Dez cenários de QR, 11 de apresentação/performance, seis de acesso público e 14 de gestão/health. Com os dez temporais: 51 cenários distintos. |
| Perfil 4G | Apresentação, capa, QR e disponibilidade prontos em 747/734/707/731/709ms; máximo 747ms, limite 3000ms. |
| Carga pública | 1004 eventos, dez clientes/60s, 264262 consultas, zero erros, p95 3,437ms, máximo 25,675ms. |

AC01/AC02: matriz confirma os intervalos [abertura, encerramento) e ausência de reabertura às 13h. AC03: estado vem do servidor e horários civis permanecem em Brasília, apesar do fuso/Date do cliente. AC04: timers/foco/online/ocultação, falha e descarte de respostas antigas cobertos sem deslocar foco. AC05: mudança da galeria não interfere e parada/retorno do serviço mantém os marcos originais.

O host temporal usa Kestrel com controller/middleware/serviço/repositório reais, o mesmo PostgreSQL DML e um arquivo privado para substituir TimeProvider. Não executa o Program de produção completo; o Compose real valida esse host separadamente. Só o GET público selecionado é encaminhado ao host temporal pelo Playwright; a jornada de indisponibilidade para/reinicia a API real através do proxy. Nenhum endpoint/configuração de produção altera hora.

Chromium 158.0.8127.18/WebKit 27.2 emulados. Perfil 4G customizado: 10/2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cinco contextos novos/cache desligado. Sem inferência sobre AWS, aparelho físico, rede comercial ou uploads. Nenhum teste ignorado/aprovado por retry automático; as execuções corrigidas foram acionadas manualmente. Gestão foi executada nos dois laboratórios de regressão e contada uma vez.

## Entrega por subtarefa

Os estados atuais são mantidos no Trello. [Transições](../../testing/2026-11-13_16-55-35/transicoes.json) usam datas UTC nativas da atividade do cartão, com actionId para rastreio. Não são horas ativas exclusivas.

| ID | Entrega/evidência |
| --- | --- |
| PP-004-001 | Enum puro SubmissionWindowStateEnum e mapeamento textual not_open/open/closed. |
| PP-004-002 | Intervalo [abertura, 12h seguinte); ticks unitários e matriz HTTP ±1ms; 13h não reabre. |
| PP-004-003 | serverTime UTC, marcos civis/timeZone e revalidateAfterMs documentados no FDD/OpenAPI. |
| PP-004-004 | Prazo máximo 30s/borda, foco/online/visibilidade e disponibilidade não confirmada na falha. |
| PP-004-005 | FDD separa captura/seleção de tentativa autorizada e tolerância de conclusão. |
| PP-004-006 | Estado público informativo; autorização/sessão de upload permanece fora da implementação. |
| PP-004-007 | SubmissionWindowPolicy chama EventPolicy.Window, a mesma regra da gestão. |
| PP-004-008 | Política pura classifica pelo instante recebido, convertendo pela BusinessClock. |
| PP-004-009 | PublicEventService lê clock.UtcNow uma vez; CountingTimeProvider verifica uma chamada. |
| PP-004-010 | America/Sao_Paulo no servidor; query com serverTime é recusada, sem confiar no cliente. |
| PP-004-011 | Models/Responses puros em Domain; nenhum campo de galeria/conteúdo/identidade acrescentado. |
| PP-004-012 | OpenAPI exige sete campos públicos e publica exemplos antes/durante/depois. |
| PP-004-013 | Cache-Control no-store existente preservado e verificado em HTTP real. |
| PP-004-014 | Adaptador valida enum, datas civis, UTC, zona e orçamento 0–30000; descarta campos extras. |
| PP-004-015 | Estado anterior mostra data/hora civil e rótulo explícito de Brasília. |
| PP-004-016 | Estado aberto informa a janela e o encerramento, sem simular confirmação de envio. |
| PP-004-017 | Estado closed mostra encerramento, inclusive durante a tolerância futura de conclusão. |
| PP-004-018 | Nome/capa/link/QR preservados antes/depois e durante falha transitória. |
| PP-004-019 | Referência serverTime e duração por performance.now; Date do aparelho em 2099 não altera estado. |
| PP-004-020 | Timer pela borda/prazo, com cleanup no unmount/troca de rota; nenhum avanço local de estado. |
| PP-004-021 | Foco/online/retorno visível coalescem chamadas; ocultação cancela timer/requisição. |
| PP-004-022 | Resposta com referência antiga é recusada; lease consumido no transporte fica a confirmar. |
| PP-004-023 | Output semântico com aria-live/atomic; transição observada na árvore acessível sem mover foco. |
| PP-004-024 | Sem captura/links sem destino; disponibilidade não cria permissão de escrita ou acesso à galeria. |
| PP-004-025 | OpeningIsInclusiveAtTickPrecision: antes/no/depois da abertura. |
| PP-004-026 | ClosingIsExclusiveAtTickPrecision: antes/no/depois de 12h. |
| PP-004-027 | CompletionToleranceNeverReopensNewSubmissions e HTTP 12h30/13h/depois. |
| PP-004-028 | Madrugada, 31/01, 31/12 e 29/02 em unitários e integração HTTP. |
| PP-004-029 | Política não recebe modo da galeria; troca immediate/scheduled pela API mantém janela. |
| PP-004-030 | Matriz de 36 HTTP GETs com provider por arquivo privado só em probes; sem endpoint de relógio. |
| PP-004-031 | Unitários e browser com Date/fuso divergentes; suspensão/timers/foco/online cobertos. |
| PP-004-032 | Edição/restauração pela API real, resposta abortada/antiga e falha de revalidação cobertas. |
| PP-004-033 | Página mantida aberta atravessa abertura/encerramento, com anúncio semântico e foco preservado. |
| PP-004-034 | Cada browser para/reinicia o service de seu Compose descartável e compara marcos sem extensão. |
| PP-004-035 | FDD, READMEs, plano/execução, revisões, matriz, capturas, falhas e histórico do Trello consolidados. |

## Falhas, dados e recuperação

[Falhas e correções](../../testing/2026-11-13_16-55-35/falhas-e-correcoes.json): propriedade incorreta no novo unitário, regra de lint do status nativo, health duplicado no host exclusivo (duas execuções sem browser) e Docker indisponível durante a última regressão. O laboratório interrompido exigiu [limpeza restrita aos próprios recursos](../../testing/2026-11-13_16-55-35/limpeza-laboratorio-interrompido.json); a suíte foi repetida após restabelecer o Docker. Não inferir horários individuais das falhas quando não registrados.

Inspeção também preservou “evento não encontrado” ao ocultar uma aba sem apresentação conhecida e reforçou o teste para aguardar a resposta anterior à borda de 12h. Versão final retestada. Capturas selecionadas de desktop/móvel foram inspecionadas; emulação e árvore semântica não substituem revisão humana/leitor real.

Snapshots HTTP de todas as tabelas preservados após leituras. Edição/restauração de fixtures grava intencionalmente evento/auditoria; demais tabelas permanecem iguais. Regressões confirmam leitura sem escrita e logs sem código/nome/URL utilizável. Nenhum pacote novo, DDL, migration ou alteração no volume persistente. Laboratórios exclusivos removidos; aplicação local parada. [Revisões](../../testing/2026-11-13_16-55-35/revisoes.json).

Publicar API antes da UI que exige a janela. Recuperação: reverter UI, depois API, sem alteração de banco. Captura, identificação/concordância, autorização/tolerância de upload, galeria, autenticação e AWS continuam em entregas próprias. Sem merge/deploy desta história.
