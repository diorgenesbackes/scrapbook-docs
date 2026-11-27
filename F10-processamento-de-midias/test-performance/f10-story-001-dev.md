# F10-STORY-001 · Carga local de preparação de fotos

Em 27/11/2026, o reteste local confirmou 80 envios e publicou as duas variantes JPEG de cada uma das 90 fotos em até 16,139 s a partir do início da rajada. Aos 14,509 s já havia 87 pares, acima dos 86 necessários para atingir 95% da coorte. A leitura posterior dos objetos validou versão, tipo, tamanho e SHA-256 dos 90 originais e das 180 variantes. O ensaio usa um relógio de fixture baseado no monotônico do host, após observar saltos no relógio civil da VM. [Resultado de carga](summary.json) · [Integridade de objetos](hash-verification.json) · [Histórico das rodadas](diagnostico-das-rodadas.md)

## Candidato e perfil

| Componente | SHA de dev |
| --- | --- |
| Serviço | `e9b6fe481d2b1cbd2a2e85fbe2ba6e60aa8dee9d` |
| UI | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| Testes | `a99d9b681788cd81f14e3c0427992f9febb86fa6` |
| Infraestrutura | `4a2c7e10f5f3b4a3982eab0b4c337a39d3b3f2f1` |
| Liquibase | `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` |

A VM Colima ARM64 descartável tinha 4 vCPU, 6 GiB de RAM e disco de 64 GiB. A stack em loopback usou Nginx, PostgreSQL 18.6, Moto com S3 versionado e SNS/SQS e duas réplicas da API. Cada réplica tinha dois workers de foto. O executor foi k6 v2.3.0 com Python 3.13.15, Docker Compose 5.6.0, Liquibase 4.33.0 e AWS CLI v2.37.10. [Ambiente e arquivos de entrada](environment.json)

Cada evento recebeu 35 depoimentos de uma foto e cinco momentos de duas fotos. A massa sintética somou 185.030.650 bytes. Foram 32 JPEGs de 1.280 × 960 (277.571 bytes), 30 de 2.400 × 1.800 (1.135.883 bytes) e 28 de 4.032 × 3.024 (5.073.996 bytes). O aquecimento com outra foto em outro evento foi excluído das métricas. O k6 iniciou 80 VUs sem rampa, com uma tentativa e consentimento próprios por convidado. Consultou o status a cada 0,9 a 2,1 s até obter o recibo, com timeout de 20 s por chamada e janela de até 105 s de consulta por tentativa.

O runner fez PUTs reais e observou a versão e o tamanho do objeto temporário antes de publicar a evidência sintética de criação no SQS local. O backend produziu os recibos da F08 e os fatos da F10 pelo fluxo normal de persistência e mensageria. A listagem de versões do Moto usou uma cópia de metadados limitada a essa operação. Depois dos degraus do relógio da VM nas primeiras rodadas, o reteste avançou a fixture de tempo da API a partir de uma âncora UTC e do monotônico do host. A amostragem do PostgreSQL e do host durante a rajada observou diferenças entre −9,344 e −4,660 ms. [Evidências de entrega](receipt-delivery.json) · [Série do PostgreSQL](database.jsonl)

O comando usa o runner temporário e os JPEGs da pasta `runner/` no laboratório da F10. A fixture controlou o relógio e o comportamento de listagem do emulador. O serviço utiliza o mesmo binário integrado em `dev`.

```sh
TMPDIR=/private/tmp DOCKER_CONTEXT=colima-scrapbook-f10-validation \
  SCRAPBOOK_F10_ADVANCING_CLOCK=1 python3 -B runner/run-photo-burst.py
```

## Recibos e latência da API

As 80 tentativas começaram em 6,1 ms, com pico de 80 em andamento. O k6 concluiu 80 iterações e emitiu 839 chamadas HTTP, com 90 PUTs e 80 recibos. Houve zero erros HTTP e zero falhas de contrato. Cada grupo abaixo apresenta percentis pelo método do posto mais próximo. O k6 também aprovou todos os limiares operacionais de p95 de 500 ms, usando sua interpolação. [Métricas HTTP](http.json)

| Operação | Amostras | p50 (ms) | p95 (ms) | p99 (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: | ---: | ---: |
| Consentimento | 80 | 91,906 | 118,428 | 129,515 | 129,515 |
| Admissão | 80 | 80,382 | 104,432 | 118,375 | 118,375 |
| Autorizar upload | 90 | 181,586 | 218,372 | 233,047 | 233,047 |
| Transferir JPEG por PUT | 90 | 280,257 | 478,263 | 489,451 | 489,451 |
| Sinalizar conclusão | 80 | 18,147 | 127,481 | 162,348 | 162,348 |
| Consultar status | 419 | 5,852 | 13,481 | 19,405 | 22,138 |
| Do início da tentativa ao recibo | 80 | 8.244 | 13.855 | 14.753 | 14.753 |

## Prontidão e recursos

O PostgreSQL registrou o timestamp de commit da confirmação e da publicação. O runner contou uma foto pronta quando os registros de imagem e miniatura estavam prontos, com referências versionadas, SHA-256 e o mesmo `xmin` da mudança do arquivo para disponível. O limiar usou a cota monotônica por arquivo, com a detecção dos commits por polling. A cota inclui a incerteza dos intervalos de polling. Nenhum arquivo ficou pendente, falhou ou exigiu nova tentativa no reteste final. O banco observou 80 registros confirmados, 90 arquivos e 180 variantes. [Resultado](readiness.json)

| Intervalo para as 90 fotos | p50 (ms) | p95 (ms) | p99 (ms) |
| --- | ---: | ---: | ---: |
| `confirmedAt` da F08 até o commit de disponibilidade | 1.136,068 | 1.642,252 | 1.889,256 |
| Commit de confirmação até commit de disponibilidade | 1.111,525 | 1.599,240 | 1.857,228 |
| Cota monotônica por arquivo | 2.323,708 | 3.428,919 | 3.435,256 |

Em 14 amostras do banco, o pico foi de seis jobs pendentes, quatro em execução, quatro arquivos aguardando admissão, três fatos ainda não publicados e 37 conexões. Ao final, os contadores de pendência ficaram em zero. As oito amostras aproximadas de SQS indicaram zero mensagens visíveis, em voo e atrasadas nas filas da F08 e F10. A frequência da coleta permite que filas transitórias surjam entre amostras. As consultas de `/tmp` registraram até 756 KiB usados por réplica, de 512 MiB disponíveis. [Série de fila](sqs.jsonl) · [Série do PostgreSQL](database.jsonl) · [Uso de armazenamento temporário](tmpfs.jsonl)

| Contêiner | Pico de CPU (% de um núcleo) | Pico de RAM (MiB) |
| --- | ---: | ---: |
| API 1 | 180,54 | 283,9 |
| API 2 | 184,58 | 198,9 |
| PostgreSQL | 48,20 | 193,1 |
| Moto | 35,08 | 460,4 |
| Nginx | 1,03 | 5,25 |

Cada contêiner teve oito amostras. Depois de medir, o runner verificou os 270 objetos referenciados no banco por leitura de suas versões privadas e encerrou com código zero. A sessão verificou a remoção dos contêineres, volumes e redes da stack, restabeleceu `systemd-timesyncd` na VM e desligou o perfil descartável. [Uso dos contêineres](containers.jsonl)

## Alcance

Os 90 arquivos atenderam a meta de prontidão em até 300 s nesse perfil local de fotos. As cinco operações de API ficaram abaixo de 500 ms de p95. O ensaio misto de 200 convidados e 170 arquivos, a configuração real da AWS e a transcodificação MediaConvert pertencem à F10-STORY-005.
