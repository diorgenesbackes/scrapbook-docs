# F08-STORY-004 — Reteste em navegador sobre dev

O runner `python3 -B video_submission_stack.py --with-browser` terminou com código 0 em 101,806 s. As seis jornadas Chromium passaram em 40,8 s, sem falhas, skips ou retries. O executor usou Node 26.13.0, npm 11.20.0, Playwright 1.65.0 e Chromium revisão 1254. [Plano de aceite](../test-funcional/2026-11-25_23-54-00/1-plano.md)

| Cenário | Resultado |
| --- | --- |
| MP4 de 60 s no momento e de 300 s no depoimento | Limites inclusivos com recibo. |
| MOV no depoimento e WebM no momento | Arquivos selecionados aceitos com legenda de 500 escalares. |
| Vídeo gravado com MediaRecorder e entrada sintética | Revisão, consentimento e recibo sem legenda. |
| Multipart interrompido na parte 2 | Retomada com a mesma tentativa e 16 MiB reconhecidos, somente as partes pendentes autorizadas e recibo após a prova de 13h. |

O runner também conferiu o status confirmado por nova consulta autorizada, a proibição de cache, os dados mínimos do recibo e a revisão do vídeo. O JUnit da execução tem SHA-256 `fd8ee11f863e598f882d628e86a5e1452ef082d419d5e7a6352df25cd28bd486`. [Aceite funcional](../test-funcional/2026-11-25_23-54-00/2-execucao.md)

A medição inicial passou em quatro jornadas. A gravação abre a revisão automaticamente e a conclusão multipart aguarda a prova de recepção após o 202. As expectativas das duas jornadas foram corrigidas e verificadas no reteste. A primeira tentativa integrada sofreu falta de espaço na preparação do PostgreSQL. O disco do perfil de validação Colima foi ampliado de 24 para 48 GiB antes da execução aprovada.

| Componente | SHA usado |
| --- | --- |
| Serviço | `203cc8365c326bf542aa08aa211f3dd5c7b3a9af` |
| UI | `b34aa71b21149f5481b199813f9b5e767c1dc3c4` |
| Infraestrutura | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |
| Testes | `1593873a095d7a1a9583775608eb8d587c63f406` |

A entrada de captura foi sintética e o laboratório usou PostgreSQL e Moto. O perfil de URL fica para o aceite em homologação, incluindo a entrega nativa da notificação S3. [Escopo publicado](../delivery/f08-story-004.md)
