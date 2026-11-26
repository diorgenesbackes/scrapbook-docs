# F08-STORY-003 · carga local no resultado integrado em dev

Ensaio de 25/11/2026, relativo ao T11. Snapshots integrados em dev com serviço `c8c7ed9`, UI `a46fc57`, testes `81d85ea`, documentação `1c0db35`, Liquibase `a7cbec2` e infraestrutura `8a0f217`. A meta é p95 de até 500 ms por operação de API, com o PUT de mídia medido à parte, e 200 envios simultâneos em dois eventos de 100. [Spec](../spec.md#7-estratégia-de-testes) · [PRD](../../PRD.md#4-objetivos)

## Prontidão com cache frio

Chromium 158.0.8127.18, Playwright 1.65.0, viewport Android de 393 × 851, toque, cache desligado, RTT 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s. A CPU do host ficou sem limitação artificial. Cada rota teve cinco navegações em contextos novos, antes da carga de HTTP. A prontidão termina nos links públicos de contribuição visíveis ou no seletor de fotos habilitado, conforme a rota. A montagem de prévias foi cronometrada desde a seleção de arquivos até todas as imagens estarem carregadas.

| Rota | Prontidão em ms, cinco amostras | Prévia em ms, cinco amostras |
| --- | --- | --- |
| Página pública do evento sintético | 2.115, 2.061, 2.064, 2.063, 2.043 | — |
| Editor de momento, dez PNGs de 922.428 bytes | 2.108, 2.075, 2.073, 2.089, 2.088 | 424, 426, 425, 421, 439 |
| Editor de depoimento, um PNG de 922.428 bytes | 2.072, 2.071, 2.091, 2.068, 2.088 | 96, 98, 97, 93, 95 |

A meta de 3 s foi atendida nas 15 sessões do perfil emulado. A medição do depoimento com foto usa a UI da 003 e registra a prontidão da entrada de foto simples da 002.

## Perfil da rajada

k6 2.3.0 no macOS com 12 núcleos lógicos. Stack descartável em loopback, com Colima ARM64, Docker 29.5.2, quatro CPUs virtuais, 6,2 GB de memória, uma réplica de API, Nginx, PostgreSQL preparado pelo Liquibase e Moto S3 versionado com SQS. O serviço preservou o relógio de sistema da VM para admissão, assinatura S3 e worker.

A rajada planejou 200 VUs em uma iteração cada, 100 em cada evento. Cada evento recebeu uma coorte de 80 momentos com uma foto e 20 com dez fotos. Os 560 arquivos planejados incluíram 280 PNGs de 922.428 bytes e 280 JPEGs de 351.104 bytes, todos sintéticos de 640 × 480, total de 356.588.960 bytes. Cada convidado usou consentimento, segredo e tentativa próprios. O gerador seguiu a ordem do cliente, com admissão de manifesto, emissão de autorização e PUT sequencial por arquivo, seguido de POST de conclusão. Usou uma janela de 75 s mais 15 s de encerramento, timeout de 15 s por requisição e zero retentativas.

As notificações nativas do Moto foram avaliadas no aquecimento inicial. O código da imagem local mostrou que o evento emitido omite `s3.object.versionId`, campo obrigatório no leitor da aplicação. O aquecimento permaneceu em `receiving` e a rajada prevista ficou preservada para a fixture manual. Na segunda stack, o aquecimento de um e dez arquivos publicou evidências com versões reais. As evidências da coorte de carga foram publicadas na fila SQS local após as chamadas medidas, usando as versões e tamanhos do armazenamento e o instante do serviço após a transferência. A fase posterior permite observar as transações do worker de processamento com sinais sintéticos.

## Resultados da rajada

As 200 admissões iniciaram em 9,1 ms, com pico estimado de 200 requisições em andamento e 100 por evento. O gerador finalizou as 200 iterações em uma janela de 2,82 s, cerca de 71 envios por segundo até o sinal de conclusão. Houve 1.720 respostas HTTP, incluindo 200 consentimentos antes da rajada, sem falha de HTTP, timeout, descarte ou retentativa. Todas as 1.520 verificações semânticas passaram. Os percentis usam interpolação linear em amostras completas de `http_req_duration`, excluído o bloqueio de conexão do cliente. O p95 de admissão incluindo esse bloqueio foi 569,553 ms.

| Operação | Respostas | p50 em ms | p95 em ms | p99 em ms | Máx. em ms | Pico estimado |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Admissão de momento | 200 × 201 | 243,396 | **567,726** | 634,003 | 653,658 | 200 |
| Autorização por arquivo | 560 × 200 | 5,745 | 323,341 | 358,298 | 383,734 | 180 |
| Transferência PUT | 560 × 200 | 195,776 | 1.242,226 | 1.347,006 | 1.403,177 | 188 |
| Sinal de conclusão da API | 200 × 202 | 3,143 | 30,554 | 43,494 | 98,115 | 8 |

| Operação | Amostras, 1 foto | p95 em ms, 1 foto | Amostras, 10 fotos | p95 em ms, 10 fotos |
| --- | ---: | ---: | ---: | ---: |
| Admissão | 160 | 567,726 | 40 | 467,564 |
| Autorização por arquivo | 160 | 354,705 | 400 | 223,409 |
| Transferência PUT | 160 | 1.324,944 | 400 | 531,637 |
| Sinal de conclusão | 160 | 30,782 | 40 | 4,154 |
| Fluxo até o sinal de conclusão | 160 | 1.852,200 | 40 | 2.808,150 |

Todos os 356.588.960 bytes planejados chegaram à fixture em 560 versões. O p95 por PUT foi 667,302 ms em 280 JPEGs e 1.242,479 ms em 280 PNGs. Os tempos por coorte refletem a ordem de envio sequencial dos arquivos. Os primeiros PUTs disputam a mesma janela e PUTs posteriores de conjuntos longos ocorrem quando a primeira onda já passou.

A meta de 500 ms falhou na admissão, com p95 de 585,750 ms no evento A e 567,119 ms no evento B. O k6 marcou a execução como falha pelo limite de admissão. Autorização e sinal de conclusão ficaram abaixo do limite. A admissão abre uma transação com `FOR UPDATE` por evento e grava os arquivos em comandos separados durante a trava. A execução não mediu o tempo de espera das travas nem isolou a contribuição de cada trecho, portanto essa é uma hipótese para investigar o excesso.

## Observação depois da transferência

Antes de publicar as evidências, o banco continha 100 admissões por evento, com a distribuição esperada de 80 conjuntos unitários e 20 de dez fotos. Os 200 registros estavam em `receiving`, com 560 arquivos declarados. A listagem do Moto confirmou o tamanho dos 560 objetos temporários da rajada e os 11 do aquecimento.

A SQS local aceitou as 560 mensagens sintéticas em 4,846 s. Após a injeção, o relatório colheu 24 amostras do banco em até 120,106 s.

| Estado ou efeito | Primeira amostra | Última amostra |
| --- | ---: | ---: |
| Conjuntos confirmados | 9 | 11 |
| Conjuntos validando | 89 | 98 |
| Conjuntos recebendo | 102 | 91 |
| Recibos S3 gravados | 317 | 351 |
| Originais preservados | 9 | 11 |
| Fatos de mídia | 9 | 11 |
| Primeiros fatos por evento, total | 2 | 2 |

Os picos e os bytes comprovam admissão e transferência para 200 participantes neste perfil. A confirmação dos 200 conjuntos ficou inconclusiva dentro da janela de observação, com 189 pendências e sem causa isolada para a drenagem parcial. A janela de dois minutos e as notificações de teste após o PUT limitam a extrapolação. O ensaio também não avaliou variantes da F10, processamento de vídeo ou áudio, carga sustentada, arquivos no máximo permitido ou os serviços AWS de homologação.

As duas stacks foram encerradas, com zero contêineres e volumes restantes e apenas as redes padrão no contexto Docker dedicado. O benchmark usou scripts temporários e os snapshots integrados, sem alterar os repositórios.

[Amostras agregadas e observação do banco](f08-story-003-dev-metricas.json)
