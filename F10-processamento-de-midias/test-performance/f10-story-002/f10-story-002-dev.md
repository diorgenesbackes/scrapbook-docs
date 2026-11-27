# F10-STORY-002 · Carga local de preparação de áudio

Em 27/11/2026, a rajada local confirmou e publicou **40 de 40 áudios**, com pico de **40 envios em andamento** e nenhuma falha. Todos os commits de prontidão foram observados em até **1,896 s** da respectiva confirmação pelo relógio monotônico, incluindo a espera pela consulta do banco. O p95 entre os commits de confirmação e prontidão no PostgreSQL foi **1,342 s**. Os oito WebM passaram pela normalização privada e chegaram à fixture como OGG. [Resultados agregados](readiness.json) · [Amostras por envio](readiness-samples.json)

## Candidato e ambiente

| Componente | SHA do checkout medido |
| --- | --- |
| Serviço | `d7a6a4ba34cbf83ed6976ea2c55713c8f3fdef1e` |
| Infraestrutura | `b7bf2f746a2122b5511f1867bc2b9d2e384a3c58` |
| Testes | `6338f1efec19c7ccbd5fe12f09e8ba7c224f9554` |
| Liquibase | `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` |
| UI | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| Documentação | `d3c9700ddef5cf1ab4b02e287b24bc99a69f5510` |

A alteração posterior de infraestrutura `b8c825736a5e065d78bd122c384ff373107d6182` modificou a política IAM do template CloudFormation e seu teste estático. A comparação confirmou que `compose.yaml` e `scripts/photo_media_fixture.py` têm o mesmo conteúdo nas duas revisões. A medição usou o SHA de infraestrutura indicado na tabela.

O host Apple M4 Pro tinha 12 núcleos e 24 GB de RAM. O perfil descartável Colima ARM64 tinha 4 vCPUs e 6 GiB de RAM. Ele executou PostgreSQL 18.6, Moto com S3 versionado, SNS/SQS e duas réplicas .NET 10.0.12, cada uma com dois workers de áudio. O Nginx recebeu as chamadas em loopback e as encaminhou à API principal. Os workers das duas réplicas executaram a preparação. O gerador usou Python 3.13.15, Docker 29.8.2, Compose 5.6.0, Liquibase 4.33.0, AWS CLI 2.37.10 com credenciais sintéticas do Moto e FFmpeg 9.0.2 no host e 9.1 no serviço. [Ambiente, versões e digests](environment.json)

## Perfil e recibos

Houve um envio de aquecimento em evento separado antes da medição. Quarenta clientes partiram de uma barreira sem rampa, com 20 envios em cada um de dois eventos. Cada evento recebeu quatro entradas de cada família. Os 40 clientes iniciaram em 4,3 ms e receberam os recibos em 3,717 s, com 10,761 recibos/s nessa rajada. A massa total foi de 2.833.800 bytes. [Operações HTTP e concorrência](http.json)

| Container e codec | Arquivos | Bytes por arquivo | Duração (s) | Prontos |
| --- | ---: | ---: | ---: | ---: |
| MP3 · MP3 Layer III | 8 | 24.467 | 3,000 | 8 |
| M4A · AAC | 8 | 27.629 | 3,000 | 8 |
| WAV · PCM | 8 | 264.678 | 3,000 | 8 |
| OGG · Vorbis | 8 | 8.669 | 3,000 | 8 |
| WebM · Opus | 8 | 28.782 | 3,008 | 8 |

Cada cliente cumpriu o fluxo normal da F08 com consentimento, admissão, autorização, PUT assinado, sinal de conclusão e consulta do recibo. Cada PUT retornou uma versão do objeto. O runner publicou as 40 provas de criação no SQS em cinco lotes. As 355 chamadas HTTP, incluindo os PUTs, terminaram sem erros ou timeouts. Os 40 recibos tinham o mesmo instante de confirmação gravado no banco e registrado pelo trabalho da F10. [Contagens por operação](http.json) · [Correlações por envio](readiness-samples.json)

| Operação | n | p50 (ms) | p95 (ms) | p99 (ms) |
| --- | ---: | ---: | ---: | ---: |
| Consentimento | 40 | 36,815 | 91,095 | 92,720 |
| Admissão | 40 | 49,898 | 92,751 | 124,723 |
| Autorizar PUT | 40 | 41,267 | 53,573 | 55,006 |
| Transferir por PUT | 40 | 26,494 | 44,469 | 44,872 |
| Sinalizar conclusão | 40 | 23,821 | 38,037 | 38,342 |
| Consultar recibo | 155 | 4,704 | 17,709 | 43,097 |
| Do início do cliente até o recibo | 40 | 2.500,654 | 3.364,185 | 3.714,715 |

## Processamento e prontidão

A espera usa `confirmedAt` da F08 até `started_at` da primeira tentativa. O intervalo seguinte termina na persistência de `provider_submit_started_at`, após a preparação da entrada. O marco de prontidão exige arquivo `available` e variante `playback` M4A/AAC `ready` com referência privada versionada, SHA-256 e o mesmo identificador de transação do arquivo. O PostgreSQL disponibiliza os instantes de commit com `track_commit_timestamp`. Todos os percentis usam o método do posto mais próximo. Para séries de 40 amostras, o p99 corresponde ao máximo. [Amostras de prontidão](readiness-samples.json)

| Intervalo | n | p50 (ms) | p95 (ms) | p99 (ms) |
| --- | ---: | ---: | ---: | ---: |
| Confirmação da F08 até início da tentativa | 40 | 567,142 | 699,788 | 735,076 |
| Início da tentativa até intenção de submissão | 40 | 58,380 | 129,131 | 141,747 |
| Commit de confirmação até commit de prontidão | 40 | 1.091,109 | 1.341,618 | 1.471,128 |
| Confirmação da F08 até commit de prontidão, intervalo estimado | 40 | 954,658–1.112,487 | 1.204,139–1.361,968 | 1.344,322–1.502,151 |
| Confirmação da F08 até observação do commit pelo host | 40 | 1.378,973 | 1.747,055 | 1.895,919 |
| Tempo interno de CreateJob na fixture | 40 | 27,145 | 63,882 | 102,834 |
| PUT do resultado e duas publicações no SQS pela fixture | 40 | 19,384 | 51,655 | 76,142 |

A fixture de relógio já versionada substituiu a injeção do TimeProvider em uma cópia temporária do serviço. Ela atualizou o arquivo compartilhado no mesmo inode aproximadamente a cada 25 ms, com uma âncora UTC e avanço pelo monotônico do host. A API e o relógio das tentativas compartilham essa base. Os oito pontos até a observação dos 40 commits, em 3,925 s, mostraram o PostgreSQL cerca de 30,3 s atrás do host. O primeiro ponto com o relógio da VM novamente próximo do host ocorreu durante a drenagem, em 9,441 s. A subtração direta entre `confirmedAt` da API e o commit SQL deu valores negativos. O intervalo estimado corrige o desvio pelo envelope das oito consultas anteriores à drenagem e assume que ele permaneceu nesse envelope entre amostras. A cota independente de 1,896 s inclui o atraso de observação e usa o monotônico do host. [Série do PostgreSQL](database.jsonl) · [Análise dos relógios](clock-analysis.json)

A fixture reutilizou o mesmo M4A/AAC de referência, gerado antes da rajada. As duas durações da fixture usam o monotônico dentro de seu handler. CreateJob inclui a leitura da entrada versionada, a gravação da saída e as duas publicações. A duração de saída mede o subconjunto de gravação e publicação. Ambas excluem a espera para entrar no lock da fixture e o transporte HTTP. [Amostras e agregados](readiness.json)

A fixture registrou 40 chamadas CreateJob e 40 jobs, publicou 80 notificações COMPLETE e recebeu 120 consultas GetJob e zero ListJobs após o aquecimento. O banco registrou 40 registros únicos do consumidor de resultados e 40 tentativas bem-sucedidas, com um job e uma variante pronta por envio. As 40 entradas privadas e 40 saídas versionadas corresponderam às referências registradas pela fixture. Os 40 originais permaneceram versionados e com o checksum registrado na F08. As consultas observaram picos de 5 jobs pendentes, 4 em execução, 13 em reconciliação, 2 tentativas em estado desconhecido e 10 arquivos confirmados aguardando admissão. Ao final, todos esses contadores eram zero, sem falha definitiva ou fato para F13. As três filas tiveram zero mensagens visíveis, em voo ou atrasadas nas amostras inicial e final. [Estados, filas e contagem de chamadas](readiness.json)

| Contêiner | Pico amostrado de CPU (% de um núcleo) | Pico amostrado de RAM (MiB) |
| --- | ---: | ---: |
| API principal | 90,19 | 170,0 |
| API secundária | 89,00 | 144,0 |
| PostgreSQL | 25,11 | 198,6 |
| Moto | 45,75 | 294,3 |
| Fixture MediaConvert | 8,77 | 42,57 |
| Nginx | 0,80 | 5,012 |

Houve cinco amostras por contêiner e 18 do banco, com pico observado de 40 conexões. Os picos são limitados à frequência de amostragem. Quatro preparos terminaram antes da rajada enquanto o runner era ajustado ao formato de data do evento, à diferença entre relógios do SigV4, ao arredondamento da assinatura e à data da prova de PUT no aquecimento. O ensaio final terminou com código zero. A limpeza confirmou zero contêineres, volumes e redes do projeto e zero contêineres das fixtures. [Amostragem dos contêineres](containers.jsonl) · [Limpeza do ensaio](cleanup.json)

## Alcance do resultado

O alvo de prontidão de 300 s foi atendido **neste recorte local de 40 áudios**, com os 40 commits observados dentro do prazo. As operações comuns de API observadas tiveram p95 abaixo de 500 ms nesse perfil. O aceite global depende da rodada mista da F10-STORY-005, com 200 envios e 170 arquivos, e de execução na AWS para medir fila, quota, transcodificação MediaConvert e integração S3/EventBridge/IAM. Os arquivos de borda de tamanho e duração e a prontidão da página em 4G ficaram fora deste ensaio. [Especificação da F10](../../spec.md) · [PRD](../../../PRD.md)
