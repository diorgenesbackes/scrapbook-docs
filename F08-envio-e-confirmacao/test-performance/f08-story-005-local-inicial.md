# F08-STORY-005 · Ensaio local inicial de carga mista

Em 26/11/2026, 200 tentativas simultâneas chegaram à API local. Os 170 PUTs foram aceitos e 105 jornadas receberam o recibo final. O emulador de armazenamento foi encerrado pelo kernel da VM por falta de memória, durante a confirmação. A admissão já havia excedido o objetivo de 500 ms antes da saturação. [Spec da F08](../spec.md#7-estratégia-de-testes) · [PRD](../../PRD.md#9-critérios-de-aceitação)

## Candidato e perfil

| Componente | SHA |
| --- | --- |
| Serviço | `224c46901c97ff13dada5e4a4dfbb3676acfb49c` |
| UI | `5c9a7dcf0c6d2e0110accd1917f4aedb0d0c73c2` |
| Testes | `2a9dffac627013175b61e23113275ac02a75bdb7` |
| Infraestrutura | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Liquibase | `4bf11b0bf07db8b07711dca47fb6e7c15e23a32e` em hlg, com árvore igual a `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` em dev |

O gerador k6 v2.3.0 no host acionou a UI Nginx e o Moto S3/SQS versionado exclusivamente por HTTP em loopback. A stack descartável Colima ARM64 tinha 4 vCPUs, 6 GiB de RAM, 48 GiB de disco, uma API .NET e PostgreSQL 18.6. Houve duas chamadas de consentimento de preparação, uma por evento, antes da rajada. Cada usuário virtual realizou uma tentativa com credencial e chave descartáveis. Após o sinal de conclusão, o k6 consultou o status em intervalos aleatórios de 1 a 2 s. A observação tinha limite de 90 s por jornada de mídia e timeout HTTP de 12 s por requisição.

Cada evento recebeu 25 depoimentos escritos, 25 depoimentos com uma foto, dez momentos com duas fotos, 20 depoimentos em áudio e 20 vídeos distribuídos entre depoimentos e momentos. Os arquivos sintéticos foram 44 PNGs de 99 bytes, 46 PNGs de 922.428 bytes, 20 MP3s de 24.467 bytes, 20 WAVs de 264.678 bytes e 40 MP4s de 285.436 bytes. A massa teve 59.636.384 bytes em 170 arquivos. O observador da fixture listou versões temporárias e publicou na SQS local envelopes sintéticos em lotes de até dez registros.

```sh
TMPDIR=/private/tmp DOCKER_CONTEXT=colima-scrapbook-f08-validation \
  python3 -B mixed_submission_load.py --output /private/tmp/scrapbook-f085-carga-inicial
```

## Resultados

As 200 tentativas começaram em 12,433 ms, com pico de 200 sobrepostas, 100 por evento. Houve 111 PUTs HTTP sobrepostos no pico estimado, em uma janela total de 0,604 s. O k6 coletou 1.713 requisições (60,42/s) e executou 200 iterações sem descarte. As 95 falhas foram consultas de estado sem resposta, com timeout de 12 s. Entre as outras consultas, 926 retornaram 200. Os percentis foram calculados por interpolação linear de `http_req_duration`, incluindo as amostras de timeout.

| Operação | Amostras | p50 (ms) | p95 (ms) | p99 (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: | ---: | ---: |
| Admissão | 200 | 258,65 | **572,30** | 603,41 | 619,37 |
| Autorização de PUT | 170 | 227,34 | 274,76 | 281,10 | 281,50 |
| Transferência PUT | 170 | 235,00 | 280,34 | 286,89 | 294,73 |
| Sinal de conclusão | 150 | 5,19 | 127,91 | 149,34 | 163,16 |
| Consulta de status | 1.021 | 3,76 | **12.000,83** | 12.001,13 | 12.001,32 |

As 105 jornadas com recibo foram 50 de texto, 30 de foto, 12 de áudio e 13 de vídeo. O tempo da admissão até o recibo teve p50 de 4,110 s, p95 de 11,066 s e máximo de 13,962 s entre as 105 jornadas concluídas. As 95 jornadas restantes não obtiveram recibo na janela executada. A última iteração encerrou em 28,321 s após o primeiro início.

A memória do Moto subiu de 134,6 MiB para 4,966 GiB. Nesse período `docker stats` perdeu amostras de Moto, PostgreSQL e API. O kernel registrou global OOM e encerrou o processo Python do Moto, com 5.197.408 KiB de RSS anônimo. A AWS CLI da fixture também excedeu o timeout de 15 s no `ListObjectVersions`, antes da captura final da contagem de publicações e do estado no banco. O runner encerrou com código 1 após 79,597 s, incluindo o ciclo da stack.

No Moto fixado (`5.2.4.dev`), a listagem de versões copia todos os objetos antes de aplicar o prefixo. A cópia duplica os buffers de conteúdo e fica retida no rastreador interno do emulador. O problema afeta tanto as buscas do observador quanto a listagem usada pela API antes de preservar cada original. O p95 de admissão foi medido nos primeiros 0,627 s e constitui uma observação independente da falha de memória.

Foram atingidas a concorrência inicial e as metas de latência para autorização e conclusão. O p95 de admissão e consulta de status e a confirmação completa da coorte ficaram fora da meta. A disponibilidade de variantes em cinco minutos pertence à F10. Homologação e prontidão do frontend em 4G exigem evidências próprias.
