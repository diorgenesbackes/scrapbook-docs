# F08-STORY-004 · Carga local de vídeo · Primeira medição

Em 25/11/2026, o k6 v2.3.0 (darwin/arm64) rodou no host e acessou a VM Colima dedicada pela interface de loopback. A VM tinha 4 vCPUs e 6 GiB configurados. O limite observado por `docker stats` foi 5,772 GiB. A API usou uma réplica, PostgreSQL 18.6 e S3 e SQS descartáveis em Moto. O runner usou Python 3.14.8 e criou dois eventos novos, com relógio de negócio controlado. O processamento do original usou o contêiner com 512 MiB de `tmpfs` para a inspeção do vídeo.

| Componente | Revisão |
| --- | --- |
| Serviço | `7f11a172ecb0056e712dca3435319a8871df6f01` |
| UI | `0a6ee94d4c72bf9c3ee00f724556193aed8d501a` |
| Liquibase | `e2c93648b9b47d33001c33a3b74dac28f9f40d22` |
| Infraestrutura | `0314b486361b4febd48255c7b6bc6a8b5494ce9b` |
| Base do runner de vídeo | `6ddba74bc7bb72970ae926336d0ef6ad0ce7b77d` |
| Script de carga | `5d2a556dd98177ee9a28daaefe2fc89fa3a78b22` |

O MP4 sintético H.264 tinha 48 MiB e 2 s, com preenchimento em uma caixa `free`. Uma execução de aquecimento antecedeu a medição. Cada tentativa enviou 16 MiB, pausou por 1 s e consultou o armazenamento pelo endpoint da API. O runner enviou apenas as partes faltantes. O observador consultou as versões do armazenamento e publicou notificações sintéticas com a versão encontrada. As latências HTTP são amostras do k6 separadas por operação. O tempo de confirmação inclui a publicação pelo observador e o polling de 1 s do cliente.

## Aquecimento

Uma tentativa confirmou com sucesso. Os três PUTs aceitaram 48 MiB em 82,6 a 89,0 ms por parte. A retomada reaproveitou 16 MiB e retransmitiu zero bytes. Entre a chamada de conclusão e o recibo decorreram 2,119 s e entre admissão e recibo decorreram 3,531 s. Foram 15 requisições sem erro, incluindo dois consentimentos de preparação.

## Retomada concorrente

Foram planejadas oito tentativas em quatro usuários virtuais, quatro por evento, com 384 MiB. O k6 observou quatro PUTs sobrepostos e cinco admissões, duas no primeiro evento e três no segundo. Doze partes aceitaram 192 MiB de 208 MiB apresentados aos PUTs. O endpoint reconheceu 64 MiB e o gerador retransmitiu zero bytes. Houve quatro notificações sintéticas de conclusão com versões encontradas no S3 local. Um PUT falhou sem status HTTP e com erro TCP k6 1200. O limite de erro abortou a rodada antes do primeiro recibo observado. A falta da janela de observação impede julgar as quatro confirmações.

| Operação | Amostras | p50 (ms) | p95 (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: | ---: |
| Admissão | 5 | 22,28 | 211,01 | 257,72 |
| Autorizar primeira parte | 5 | 23,21 | 57,91 | 66,58 |
| Autorizar parte restante | 8 | 19,07 | 25,35 | 26,67 |
| Consultar partes | 8 | 14,30 | 27,23 | 27,81 |
| Concluir transporte | 1 | 212,80 | 212,80 | 212,80 |
| Consultar status | 1 | 4,06 | 4,06 | 4,06 |
| Transferir parte | 13 | 125,92 | 384,22 | 387,86 |

A amostra de PUT inclui a falha com duração HTTP de zero. O pico observado no contêiner da API foi de 125,5 MiB e 52,22% de uma CPU. Moto registrou 334,4 MiB e 109,86% e PostgreSQL registrou 51,72 MiB e 3,00%. Os p95 abaixo de 500 ms são descritivos neste ensaio abortado.

## Admissões sem upload

Quarenta usuários virtuais executaram cinco admissões cada, 100 requisições por evento. Foram observadas 40 requisições sobrepostas e 200 chamadas em 0,382 s, com 11 respostas 201 (cinco no evento A e seis no B) e 189 respostas 503. O p50 foi 73,05 ms, p95 de 98,74 ms, p99 de 112,65 ms e máximo de 128,42 ms, incluindo as falhas. O contrato da admissão sob esse perfil falhou. A causa dos HTTP 503 não foi isolada nesta medição. A API chegou a 163,3 MiB e 82,61% de uma CPU no período observado. O PostgreSQL chegou a 53,06 MiB e 79,76%.

A primeira preparação falhou antes de gerar tráfego. A medição acima usou `TMPDIR=/private/tmp`, montado pela VM. A stack descartável encerrou o próprio ambiente. A cobertura pendente inclui 200 uploads completos com o mix do PRD, o ambiente publicado e a disponibilidade das variantes em até cinco minutos.
