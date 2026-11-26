# F08-STORY-005 · Carga mista no candidato integrado em dev

O reteste local de 26/11/2026 confirmou 200 envios simultâneos, com 100 em cada um dos dois eventos sintéticos. Todos os contratos verificados pelo gerador passaram. A admissão foi a operação de API com maior p95, de 434,345 ms. O recibo final chegou em até 12,852 s. [Spec da F08](../spec.md#7-estratégia-de-testes) · [Critérios do PRD](../../PRD.md#9-critérios-de-aceitação)

## Candidato e perfil

| Componente | SHA executado |
| --- | --- |
| Serviço | `cb102f4a55a8aeaf7a5e73a77561407e6193152a` |
| UI | `e3b71a564b634f0bd49cab842def52224059a310` |
| Testes | `51f4ca153c9f58469604546123c8c7aecab3f6a1` |
| Infraestrutura | `006e860c2fb5e78b8942226398ab2bc53590ed15` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |

O k6 v2.3.0 no host ARM64 executou 200 usuários virtuais com uma tentativa cada, sem rampa. O runner Python 3.13.15 usou a VM Colima ARM64 reservada, com 4 vCPUs, 6 GiB de RAM e disco de 48 GiB. A stack descartável em loopback manteve uma API, Nginx, PostgreSQL 18.6 e Moto 5.2.4.dev para S3 versionado e SQS. A AWS CLI v2.37.10 publicou os recibos da fixture local com a versão e o horário de cada objeto temporário. O relógio de negócio foi fixado pela fixture durante a rodada.

Cada evento recebeu 25 depoimentos de texto, 25 depoimentos de uma foto, dez momentos de duas fotos, 20 depoimentos em áudio e 20 vídeos distribuídos entre depoimentos e momentos. O payload dos 170 arquivos somou 59.636.384 bytes. Foram usados 44 PNGs de 99 bytes, 46 PNGs de 922.428 bytes, 20 MP3s de 24.467 bytes, 20 WAVs de 264.678 bytes e 40 MP4s de 285.436 bytes. Os vídeos deste perfil couberam em PUT único. Todos os arquivos eram sintéticos. Cada tentativa utilizou sua própria chave de idempotência e credencial descartável.

```sh
TMPDIR=/private/tmp DOCKER_CONTEXT=colima-scrapbook-f08-validation \
  python3 -B mixed_submission_load.py --output /private/tmp/scrapbook-f085-dev-carga
```

O runner preparou dois consentimentos antes da rajada. O reteste manteve a mesma preparação, mistura, recursos, limiares e timeouts do ensaio inicial. O k6 consultou status em intervalos de 1 a 2 s, com observação de 90 s por tentativa de mídia e timeout HTTP de 12 s. A instrumentação complementar leu os logs de API e Nginx depois de encerradas as medições.

## Latência, concorrência e integridade

As 200 tentativas começaram em uma janela de 12,096 ms. O pico observado foi de 200 tentativas em andamento, 100 por evento, com pico estimado de 112 PUTs HTTP sobrepostos. O k6 concluiu 200 iterações em 12,860 s a partir do primeiro início. Foram 1.432 chamadas HTTP, incluindo os dois consentimentos, com taxa de 111,06 chamadas/s, zero falhas ou timeouts e 890 de 890 verificações de contrato aprovadas. Os percentis usam interpolação linear das amostras de `http_req_duration`.

| Operação | Amostras | p50 (ms) | p95 (ms) | p99 (ms) | Máximo (ms) |
| --- | ---: | ---: | ---: | ---: | ---: |
| Admitir tentativa | 200 | 251,923 | **434,345** | 585,548 | 586,313 |
| Autorizar PUT | 170 | 219,998 | **273,460** | 277,860 | 283,015 |
| Sinalizar conclusão | 150 | 5,051 | **116,772** | 130,921 | 155,573 |
| Consultar status | 740 | 3,101 | **6,081** | 7,489 | 13,858 |
| Transferir arquivo por PUT | 170 | 199,347 | 266,182 | 272,479 | 273,281 |

A transferência durou 0,508 s entre o primeiro início e o último PUT concluído. Para a mesma coorte de 200 admissões, o p95 interno da API foi de 342,224 ms e o do Nginx foi de 400,900 ms. O limite de 500 ms se aplica às quatro operações medidas da API. As duas chamadas de consentimento de preparação também responderam HTTP 200, em 4,448 e 24,204 ms.

Da admissão ao recibo, a coorte completa de 200 tentativas teve p50 de 6,591 s, p95 de 12,052 s, p99 de 12,757 s e máximo de 12,852 s. O p95 dessa jornada por formato foi de 0,505 s para texto, 11,538 s para foto, 12,071 s para áudio e 12,725 s para vídeo. A duração inclui a transferência, a observação da versão S3, o consumo da fila, a preservação do original e a espera até a consulta seguinte para as mídias. O banco confirmou a mistura esperada por evento. A fixture publicou 170 notificações e o banco registrou 170 fatos de mídia e dois fatos de primeiro registro.

Em 24 amostras por contêiner, o Moto atingiu 186,4 MiB e 23,03% de uma CPU. A API atingiu 207,5 MiB e 147,94%, e o PostgreSQL atingiu 177,1 MiB e 56,74%. O runner encerrou com código zero em 37,396 s, incluindo o ciclo de preparação e limpeza dos recursos que criou.

## Correção e alcance

O ensaio anterior chegou a 4,966 GiB de memória no Moto e foi interrompido pelo kernel da VM. A adaptação exclusiva da fixture evita que cada listagem mantenha uma cópia adicional dos bytes das versões no rastreador interno do Moto. O smoke na imagem fixada repetiu 40 listagens sem acumular instâncias, verificou o prefixo, a paginação, a ordem das versões, os tamanhos e os ETags e confirmou que a cópia e a leitura por `VersionId` preservam o conteúdo. Os nove testes de infraestrutura passaram. As árvores do serviço, UI e testes do reteste são iguais às árvores correspondentes do ensaio inicial. O p95 inicial de admissão de 572,304 ms ocorreu antes da falta de memória, e a variação observada entre as duas rodadas de admissão permanece sem causa confirmada. [Ensaio inicial](f08-story-005-local-inicial.md)

As metas de concorrência e latência da API foram atendidas no perfil local e toda a coorte recebeu confirmação. A disponibilidade de variantes em cinco minutos pertence à F10. Prontidão do frontend em 4G, volumes máximos e comportamento no ambiente AWS dependem de seus respectivos ensaios. [Métricas agregadas](f08-story-005-dev-metricas.json)
