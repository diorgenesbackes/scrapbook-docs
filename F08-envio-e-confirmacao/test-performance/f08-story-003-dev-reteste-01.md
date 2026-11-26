# F08-STORY-003 · primeiro reteste da carga local

Reteste do T11 em 25/11/2026 com o serviço integrado em `47a671b7b8fb32888d53cbcee1dc349b47c2c204`. A correção consolidou a persistência do manifesto e dos objetos no repositório. UI `a46fc57`, testes `81d85ea`, docs `1c0db35`, Liquibase `a7cbec2` e infraestrutura `8a0f217` são as mesmas árvores do ensaio inicial. Os scripts k6 e sua configuração foram reproduzidos byte a byte. As medições de prontidão já registradas continuam vinculadas à mesma UI. [Ensaio inicial](f08-story-003-dev.md) · [Spec](../spec.md#7-estratégia-de-testes)

O gerador k6 2.3.0 no host fez uma rajada de 200 VUs, 100 por evento. Em cada evento houve 80 momentos de uma foto e 20 de dez. PNGs de 922.428 bytes e JPEGs de 351.104 bytes somaram 356.588.960 bytes. Configuração mantida com uma API, Nginx, PostgreSQL e Moto S3/SQS em loopback na VM Colima ARM64 com 4 vCPUs e 6,2 GB. A API usou o relógio de sistema da VM. Aquecimentos de um e dez arquivos confirmaram com notificações sintéticas antes da rajada.

## Resultado comparável

As 200 admissões começaram em 9,393 ms com pico estimado de 200, 100 por evento. Os 200 VUs concluíram o envio e o sinal HTTP em 2,436 s. Houve 200 admissões HTTP 201, 560 autorizações HTTP 200, 560 PUTs HTTP 200 e 200 sinais HTTP 202. As 1.520 verificações de contrato passaram e não houve falhas, descartes ou retentativas entre as 1.720 chamadas, incluindo os 200 consentimentos de preparação.

| Operação | Amostras | p50 em ms | p95 inicial em ms | p95 do reteste em ms | p99 em ms | Máximo em ms |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Admissão | 200 | 223,816 | 567,726 | **503,028** | 530,322 | 539,476 |
| Autorização | 560 | 4,955 | 323,341 | 279,724 | 320,067 | 336,616 |
| Transferência PUT | 560 | 145,410 | 1.242,226 | 611,880 | 645,091 | 1.180,685 |
| Sinal de conclusão | 200 | 2,925 | 30,554 | 53,842 | 61,650 | 75,075 |

| Operação | Amostras, 1 foto | p95 em ms, 1 foto | Amostras, 10 fotos | p95 em ms, 10 fotos |
| --- | ---: | ---: | ---: | ---: |
| Admissão | 160 | 513,713 | 40 | 441,250 |
| Autorização | 160 | 317,162 | 400 | 222,289 |
| Transferência PUT | 160 | 637,564 | 400 | 495,696 |
| Sinal de conclusão | 160 | 55,800 | 40 | 4,809 |

O p95 de admissão caiu 64,698 ms nessa comparação de uma rajada por candidato. O reteste ficou 3,028 ms acima de 500 ms. Por evento foram 503,378 ms e 503,028 ms. O k6 retornou 99 pelo limiar de admissão. O p95 usa interpolação linear nas amostras completas de `http_req_duration`. Incluindo espera de conexão do cliente, o p95 da admissão foi 503,848 ms. Os PUTs são medidos separadamente da meta da API. A diferença de PUT entre as duas rajadas também pode refletir a variação do ambiente local.

## Persistência e confirmação

O banco mostrou 200 registros em `receiving`, 100 por evento, e 560 arquivos antes das notificações sintéticas. A fixture S3 preservou as 560 versões da rajada com tamanho esperado, além das 11 versões do aquecimento. Os 200 sinais HTTP 202 e a consulta ao banco registraram os conjuntos à espera dos recibos.

Na etapa posterior, o comando local `sqs send-message-batch` retornou código 255 e interrompeu a coleta antes da drenagem. O helper registra o código e descarta o stderr, então a causa e a quantidade de mensagens entregues nessa etapa ficaram indeterminadas. O resultado da confirmação em coorte fica inconclusivo. O contexto Colima terminou com zero contêineres e volumes e apenas as redes padrão.

[Métricas agregadas do reteste](f08-story-003-dev-reteste-01-metricas.json)
