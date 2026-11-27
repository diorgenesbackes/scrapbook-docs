# F10-STORY-003 — Rajada local de preparação de vídeos

Em 27/11/2026, 18 convidados sintéticos enviaram vídeos simultaneamente em dois eventos no candidato integrado em `dev`. A F08 confirmou os 18 envios e a F10 disponibilizou as 18 variantes privadas do perfil 1. O p95 da confirmação ao início da tentativa foi de **664 ms** e o p95 da confirmação à primeira observação de prontidão foi de **1.165,605 ms**. No ensaio, cada arquivo teve uma tentativa e uma única variante pronta. [Cartão](https://trello.com/c/HesFEO9d) · [Spec da F10](../../spec.md#aceite-rastreado-ao-prd) · [Resultados agregados](readiness.json)

## Candidato e protocolo

- Serviço `78abe427054ee420cab27da00b1a772534e390b9`, infraestrutura `1578ca0db337c58b0ce546e4ea252b89527443f1`, testes `e19c13af9726e5605b3f7595e6a7d0637e11ecd1` e docs `99044c55707eeab1f0539e5adab523d19e335734`.
- Liquibase `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` e frontend `57f4e63f9a6378ecde532d53b4d204ac935021ff`.
- Stack descartável em loopback no Colima dedicado, arquitetura aarch64, 4 vCPUs e 5,77 GiB. Duas réplicas do serviço .NET 10.0.12, com duas vagas de preparação e validação de áudio/vídeo por réplica. PostgreSQL 18.6 migrado via Liquibase 4.33.0. O nginx encaminhou as chamadas HTTP à API principal. O Moto forneceu S3 versionado, SNS e SQS. Execução em Python 3.13.15, FFmpeg 9.0.2 no host e FFmpeg 9.1 no serviço. [Ambiente e binários](environment.json)
- Um vídeo em um terceiro evento aqueceu o fluxo antes da medição. Cada cliente da coorte realizou consentimento, admissão, autorização, PUT dos bytes, pedido de conclusão e consulta de recibo. Os 18 PUTs ficaram simultaneamente em voo e seus inícios se concentraram em 1,677 ms. Os avisos S3 da fixture seguiram em dois envelopes. A janela de observação permitia acompanhar pendências por até 300 s após as últimas respostas dos clientes. [HTTP e simultaneidade](http.json)
- O adaptador de contrato validou os parâmetros do job, leu a entrada privada por tentativa, gravou o MP4 de referência gerado com FFmpeg fora da medição e entregou dois resultados por job. O serviço verificou as saídas versionadas antes de marcar prontidão. [Entradas e catálogo da fixture](environment.json)

| Entrada | Arquivos | Bytes por arquivo | Duração da fonte | Propriedades da fonte | Variante observada |
| --- | ---: | ---: | ---: | --- | --- |
| MP4 paisagem | 6 | 1.403.204 | 6 s | 640 × 360, H.264 a 25 fps, AAC mono | 640 × 360 a 25 fps com AAC mono |
| MOV retrato | 6 | 1.326.210 | 9 s | 480 × 270 com rotação de 90°, H.264 a 24 fps, sem áudio | 270 × 480 a 24 fps, sem áudio |
| WebM quadrado | 6 | 4.326.241 | 12,008 s | 480 × 480, VP9 a 60 fps, Opus mono | 480 × 480 a 30 fps com AAC mono |

A distribuição foi de três arquivos de cada formato por evento, totalizando 42.333.930 bytes transferidos em PUT simples. Os 18 clientes receberam seus recibos em uma janela de 2,269 s, equivalente a 7,93 recibos/s no intervalo da rajada. [HTTP e simultaneidade](http.json) · [Amostras por arquivo](readiness-samples.json)

## Latências medidas

| Marco | Amostras | p50 (ms) | p95 (ms) | p99 (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: | ---: | ---: |
| PUT de vídeo | 18 | 134,713 | 204,854 | 204,854 | 204,854 |
| Início da jornada → recibo observado | 18 | 1.455,877 | 2.268,677 | 2.268,677 | 2.268,677 |
| Confirmação → início da tentativa | 18 | 402 | 664 | 664 | 664 |
| Tentativa → intenção persistida de submissão | 18 | 139 | 276 | 276 | 276 |
| Confirmação → primeira consulta com variante pronta | 18 | 993,068 | 1.165,605 | 1.165,605 | 1.165,605 |

A preparação foi observada por consulta ao PostgreSQL em intervalos de 250 ms mais o tempo da consulta. As consultas levaram entre 72,069 e 104,817 ms. A confirmação e a tentativa usam o mesmo DevClock, ancorado no monotônico do host e atualizado no mesmo inode. O intervalo máximo observado entre escritas foi 39,126 ms. A primeira observação inclui o atraso de amostragem. A última observação anterior à prontidão produziu limites inferiores por arquivo, com p50 de 533,997 ms e p95 de 713,518 ms. Os cálculos usam o monotônico do host e o DevClock. Os percentis usam nearest rank. Nesta amostra de 18 registros, p95 e p99 caem na maior observação. [Série do PostgreSQL](database.jsonl) · [Intervalos por arquivo](readiness-samples.json)

| Entrada | Prontos | p50 confirmação → observação (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: |
| MP4 | 6/6 | 913,414 | 1.118,414 |
| MOV | 6/6 | 1.024,414 | 1.088,414 |
| WebM | 6/6 | 1.051,068 | 1.165,605 |

## Consistência e recursos

- Todos os 18 recibos corresponderam ao instante persistido. Havia nove arquivos `available` por evento, com contexto de confirmação preservado, original versionado e uma variante `playback` do perfil 1. As 18 variantes ficaram prontas na mesma transação que alterou o estado do arquivo. A versão e a soma de verificação de entrada e saída coincidiram com a fixture e a persistência, e dimensões, fps, duração e codec coincidiram com o perfil esperado. [Resultados agregados](readiness.json) · [Amostras por arquivo](readiness-samples.json)
- Houve 18 chamadas `CreateJob`, 36 mensagens de resultado e zero chamadas extras de criação ou novas tentativas, falhas de cliente, falhas definitivas ou fatos de falha F13. No pico amostrado houve quatro trabalhos em execução, oito pendentes e três arquivos confirmados aguardando admissão. As três filas estavam vazias ao fim. [Resultados agregados](readiness.json) · [Série do PostgreSQL](database.jsonl)
- Em três amostras do Docker, os máximos de CPU foram 115,60% na API principal, 124,40% na secundária, 27,13% no PostgreSQL, 42,94% no Moto e 8,48% na fixture. No Docker, 100% corresponde a um núcleo. Memória amostrada até 151,6 MiB na API principal, 113,9 MiB na secundária, 136 MiB no PostgreSQL e 236,9 MiB no Moto. Estes valores são máximos da amostragem da rajada. [Amostras de contêineres](containers.jsonl)
- O executor encerrou com código 0 e confirmou zero contêineres, volumes e redes descartáveis remanescentes. Uma tentativa anterior parou antes da coorte por usar horário fixo no aviso S3 de aquecimento. O runner temporário passou a usar o instante corrente no aviso, e o aquecimento concluiu na execução reportada. [Limpeza](cleanup.json) · [Diagnóstico do aquecimento](setup-warmup/diagnostico.md)

## Alcance

O ensaio confirma o fluxo F08 → F10, o contrato de perfis MP4, MOV e WebM e a publicação versionada com resultados duplicados sob a carga local registrada. Os 18 arquivos ficaram prontos dentro da janela de 300 s neste perfil. A rajada não incluiu VFR, multipart, mídia nos limites, reinício, falha definitiva ou carga mista. O aceite global de ≥95% em 5 min requer o perfil misto de 200 envios e os serviços AWS. O desempenho do MediaConvert, a adaptação de toda a matriz de entradas e as propriedades dos artefatos produzidos pelo serviço gerenciado permanecem para homologação. [Critérios do PRD](../../../PRD.md#f10-processamento-de-m%C3%ADdias-2) · [Aceite da F10](../../spec.md#aceite-rastreado-ao-prd)
