# F08-STORY-003 · segundo reteste da carga local

Reteste do T11 em 25/11/2026 no serviço integrado `d1bac2681359924ece865e302ef3b982db8f89d0`. A admissão de fotos passou a usar trava compartilhada por evento e trava transacional por chave da tentativa. A persistência em lote da rodada anterior foi preservada. A UI `a46fc57` e os snapshots de testes `81d85ea`, docs `1c0db35`, Liquibase `a7cbec2` e infraestrutura `8a0f217` permaneceram iguais. A prontidão em 4G continua documentada no ensaio inicial dessa UI. [Ensaio inicial](f08-story-003-dev.md) · [Primeiro reteste](f08-story-003-dev-reteste-01.md) · [Spec](../spec.md#7-estratégia-de-testes)

O k6 2.3.0 repetiu o script e a configuração das rodadas anteriores byte a byte, em uma rajada de 200 VUs com uma tentativa cada e 100 em cada evento. Cada evento combinou 80 conjuntos de uma foto e 20 conjuntos de dez. Os 560 arquivos alternaram PNG de 922.428 bytes e JPEG de 351.104 bytes, total de 356.588.960 bytes. A API manteve o relógio da VM. A stack de loopback manteve uma réplica de API, Nginx, PostgreSQL e Moto versionado com SQS em Colima ARM64 de 4 vCPUs e 6,2 GB. O aquecimento de um e dez arquivos confirmou com recibos sintéticos.

## API e transferência

As 200 admissões iniciaram em 19,975 ms, com pico estimado de 200 em andamento e 100 em cada evento. O pico estimado de PUTs em andamento foi 197. As 200 tentativas completaram seus PUTs e o sinal HTTP em uma janela de 2,337 s. O k6 encerrou com status **0**. Houve 1.720 respostas HTTP, incluindo os 200 consentimentos anteriores à rajada, sem falhas, timeouts, descarte ou retentativa. Todas as 1.520 verificações de contrato passaram. Os percentis usam interpolação linear das amostras de `http_req_duration`. O p95 de admissão incluindo bloqueio do cliente foi 223,758 ms.

| Operação | Respostas | p50 em ms | p95 em ms | p99 em ms | Máximo em ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| Admissão | 200 × 201 | 158,545 | **222,288** | 229,921 | 235,277 |
| Autorização | 560 × 200 | 5,280 | 169,293 | 187,950 | 200,691 |
| Transferência PUT | 560 × 200 | 143,994 | 1.208,892 | 1.276,999 | 1.298,957 |
| Sinal de conclusão | 200 × 202 | 2,916 | 53,807 | 71,516 | 88,708 |

| Operação | Amostras, 1 foto | p95 em ms, 1 foto | Amostras, 10 fotos | p95 em ms, 10 fotos |
| --- | ---: | ---: | ---: | ---: |
| Admissão | 160 | 221,608 | 40 | 229,180 |
| Autorização | 160 | 182,399 | 400 | 152,592 |
| Transferência PUT | 160 | 1.266,845 | 400 | 507,042 |
| Sinal de conclusão | 160 | 58,938 | 40 | 5,356 |

O p95 de admissão passou de 567,726 ms no ensaio inicial a 503,028 ms no primeiro reteste e 222,288 ms no segundo. Neste reteste, os p95 por evento foram 222,999 ms e 220,529 ms. A meta de p95 de até 500 ms foi atendida nas três operações de API deste perfil. O PUT permanece uma medida separada. Os 560 PUTs criaram 560 versões do tamanho esperado, conferidas no armazenamento antes da etapa de publicação manual. O banco conferiu 100 registros por evento, 560 arquivos e zero fatos antes dos recibos.

## Observação depois da rajada

A fixture publicou os recibos sintéticos na fila local após a medição. A CLI recebeu aceite de 470 das 560 mensagens. Quatro lotes tiveram conexão encerrada sem resposta e ficaram com entrega indeterminada. Cinco lotes foram repetidos uma única vez somente após falha de conexão antes do envio, e continuaram indisponíveis. Os eventos do Docker registraram `oom` e saída 137 do contêiner Moto durante essa etapa. A falha em memória da fixture aconteceu após a conclusão da rajada de HTTP e PUT.

Dos 200 conjuntos, 11 confirmaram, 43 ficaram em validação e 146 ficaram em recebimento. O banco registrou 171 recibos, 11 originais preservados, 11 fatos de mídia e dois primeiros fatos. Esses números permaneceram estáveis nas 25 amostras após a publicação, observadas por 120,111 s. A confirmação da coorte ficou bloqueada pelo emulador. A meta de prontidão de variantes da F10 e o comportamento em serviços AWS de homologação permanecem fora deste ensaio.

O runner global retornou 1 pelos testes de publicação da coorte e confirmação. O k6 preservou o status 0 da rajada. A verificação final do contexto dedicado encontrou zero contêineres, zero volumes e apenas as redes padrão.

[Métricas agregadas, estados e eventos do Docker](f08-story-003-dev-reteste-02-metricas.json)
