# F08-STORY-004 · Carga local de retomada de vídeo

O reteste de 26/11/2026 confirmou os oito vídeos de 48 MiB e reaproveitou 128 MiB sem retransmissão. Todas as operações da API medidas atingiram p95 de até 500 ms neste perfil. A rodada separada admitiu 200 tentativas, com pico de 40 requisições sobrepostas, 100 admissões por evento e sem transferência de mídia. [Spec da F08](../spec.md) · [Critérios de performance do PRD](../../PRD.md#9-critérios-de-aceitação)

## Candidato e perfil

| Componente | SHA executado |
| --- | --- |
| Serviço com FFmpeg 9.1 | `203cc8365c326bf542aa08aa211f3dd5c7b3a9af` |
| UI | `b34aa71b21149f5481b199813f9b5e767c1dc3c4` |
| Testes | `1593873a095d7a1a9583775608eb8d587c63f406` |
| Infraestrutura | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |

O k6 v2.3.0 rodou no host e acessou em loopback a VM Colima dedicada, com 4 vCPUs, 6 GiB de memória e disco de 48 GiB. A stack descartável usou uma réplica da API, PostgreSQL 18.6, armazenamento versionado e fila Moto. O relógio de negócio foi fixado pela fixture. O Python 3.14.8 executou o mesmo runner da linha de base, com diagnóstico sanitizado de falhas. A API e o armazenamento foram observados com `docker stats`.

```sh
TMPDIR=/private/tmp DOCKER_CONTEXT=colima-scrapbook-f08-validation \
  python3 -B video_submission_load.py --output /private/tmp/scrapbook-f084-carga
```

O cenário fechado executou um aquecimento de um vídeo e uma medição de quatro usuários virtuais com duas tentativas novas cada, divididas entre dois eventos. O vídeo H.264 sintético tinha 2 s e 48 MiB por arquivo, incluindo preenchimento em uma caixa MP4 `free`. Cada tentativa enviou 16 MiB, pausou por 1 s, consultou as partes reconhecidas e enviou somente as duas restantes. Um observador consultou versões no Moto e publicou uma notificação sintética para cada versão concluída. A API confirmou pelo fluxo de consumo da fila, validação e preservação do original. A medição de tempo até o recibo inclui essa publicação e o polling de 1 s do cliente.

## Retomada e confirmação

O aquecimento confirmou um vídeo, em 2,128 s da chamada de conclusão ao recibo e 3,540 s da admissão ao recibo. Na medição, ocorreram oito admissões, 24 PUTs aceitos, oito notificações sintéticas e oito confirmações. Quatro tentativas ocorreram em cada evento. Foram 384 MiB apresentados e aceitos, 128 MiB reconhecidos na retomada e zero bytes retransmitidos. O pico observado foi de quatro PUTs sobrepostos. A janela entre o início do primeiro e o término do último PUT durou 7,226 s, com taxa agregada de 53,1 MiB/s incluindo os intervalos entre PUTs. Foram 118 chamadas HTTP sem falha, incluindo duas chamadas de consentimento de preparação.

| Operação | Amostras | p50 (ms) | p95 (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: | ---: |
| Admitir | 8 | 12,23 | 92,64 | 98,75 |
| Autorizar primeira parte | 8 | 23,75 | 29,75 | 31,61 |
| Autorizar parte restante | 16 | 8,32 | 27,19 | 45,62 |
| Consultar partes | 16 | 13,93 | 20,13 | 21,86 |
| Concluir transporte | 8 | 114,01 | 137,01 | 142,07 |
| Consultar status | 36 | 3,28 | 9,37 | 11,86 |
| Transferir uma parte | 24 | 130,26 | 224,96 | 227,01 |

O tempo da chamada de conclusão ao recibo teve p50 de 3,626 s, p95 de 4,772 s e máximo de 5,108 s. Da admissão ao recibo, p50 de 5,184 s, p95 de 6,292 s e máximo de 6,543 s. Os percentis de amostras pequenas descrevem esta rodada. A API atingiu 141,3 MiB e 40,43% de uma CPU em 32 amostras. PostgreSQL atingiu 50,76 MiB e 3,98%. Moto atingiu 892,4 MiB e 129,39%.

## Controle de admissão

Quarenta usuários virtuais fizeram cinco admissões novas cada. As 200 requisições terminaram em 0,386 s, todas com HTTP 201. A admissão atingiu p50 de 27,05 ms, p95 de 273,16 ms, p99 de 274,39 ms e máximo de 278,79 ms. O banco registrou 200 sessões em recebimento depois do perfil e preservou as nove confirmações das fases anteriores. O log do PostgreSQL registrou zero erros com SQLSTATE. Nesta fase, a API atingiu 141 MiB e 37,89% de uma CPU e o PostgreSQL atingiu 160,3 MiB e 36,68%, em seis amostras.

## Resultado e alcance

- Retomada, preservação das partes, confirmação e p95 de até 500 ms por operação de API atendidos no perfil local executado. A transferência PUT foi medida separadamente.
- O candidato integrado completou ambas as rodadas com zero erros. A primeira medição registrou um erro TCP e 189 respostas HTTP 503. A causa inicial permaneceu indeterminada. O reteste usou as mesmas cargas e limites, com candidato e instrumentação atualizados.
- Os 200 envios de mídia simultâneos com a mistura de tipos e tamanhos do PRD, a disponibilidade de variantes em até cinco minutos e o ambiente publicado permanecem pendentes de ensaio próprio. Esta rodada mediu recibos de confirmação de recebimento.
- O runner saiu com código zero em 55,076 s, incluindo o ciclo da stack, e encerrou os recursos descartáveis. Os 13 testes dos guardas de configuração passaram com 100% de linhas e branches do módulo.

[Primeira medição](f08-story-004-local-inicial.md)
