# Validação local da F10-STORY-001

Os quatro critérios da story passaram no candidato integrado em `dev`. A execução principal concluiu em 117,980 s, com quatro subcenários de integração e quatro jornadas de navegador aprovados. A verificação funcional suplementar concluiu em 31,283 s. Ambos os comandos retornaram código 0. O JUnit registra quatro casos de navegador, zero falhas, zero erros e zero skips. [Cartão da story](https://trello.com/c/3tAcFFKS) · [JUnit sanitizado](e2e-junit.xml)

## Candidato e ambiente

| Repositório | SHA |
| --- | --- |
| `scrapbook-service` | `e9b6fe481d2b1cbd2a2e85fbe2ba6e60aa8dee9d` |
| `scrapbook-ui` | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| `scrapbook-tests` | `a99d9b681788cd81f14e3c0427992f9febb86fa6` |
| `scrapbook-liquibase` | `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` |
| `scrapbook-infrastructure` | `4a2c7e10f5f3b4a3982eab0b4c337a39d3b3f2f1` |
| `scrapbook-docs` | `616cdc7359eec66f260040da78cdba2bb73368d3` |

A execução principal ocorreu em 27/11/2026, das 11h51min34s às 11h53min32s no horário de Brasília. O suplemento usou outro laboratório, das 11h54min02s às 11h54min33s. Os laboratórios usaram Docker Compose 5.6.0 no contexto Colima, dois contêineres do serviço ASP.NET 10.0.12, UI integrada, PostgreSQL 18.6 com migrations Liquibase e Moto para S3 versionado, SNS e SQS. A massa foi sintética, o relógio da aplicação foi controlado e o runner descartou seus recursos ao encerrar. Python 3.13.15, Node 26.13.0, npm 11.20.0, Playwright 1.65.0 e FFmpeg/ffprobe 9.0.2. [Spec da F10](../spec.md) · [Plano da F10](../plan.md)

## Resultados por critério

| Critério | Resultado esperado | Evidência observada | Status |
| --- | --- | --- | --- |
| CA1 / T09 · F08 → F10 e prazo (PRD F08 e F10) | Recibo antes do preparo, um trabalho por foto válida e perfil, zero trabalho para texto e conjunto rejeitado, convergência após o prazo. | Um momento de duas fotos e um depoimento HEIC geraram três trabalhos. A validação correlacionou cada trabalho ao fato `record.media-confirmed.v1` e aos dados de confirmação. Na fila com atraso, a API retornou recibo `confirmed` enquanto a contagem de trabalhos daquele arquivo ainda era zero. Nas consultas e reenvios da mesma tentativa, o recibo permaneceu idêntico e a consulta retornou `Cache-Control: no-store`. No evento de texto e conjunto rejeitado, a contagem de fatos de mídia e de trabalhos foi `0\|0`. Nas duas jornadas de câmera emulada, a tentativa obteve confirmação às 13h01 de Brasília, após evidência de recebimento às 13h00. As fotos ficaram prontas. | Aprovado |
| CA2 / T09 · Variantes (PRD F10 e HLD) | Gerar dois JPEGs decodificáveis, preservando proporção, orientação e fotos pequenas, dentro de 2.048 e 400 px, com remoção de dados de localização. | O JPEG com orientação e GPS resultou em 1.024 × 2.048 e 200 × 400. A análise dos pixels confirmou a rotação. O PNG transparente resultou em 128 × 64 em ambas as variantes, com fundo branco. O HEIC e o HEIF genérico `mif1` resultaram em 32 × 32 nas duas variantes. `ffprobe` identificou `mjpeg`, o FFmpeg decodificou ambas as saídas e as asserções descartaram segmentos EXIF, XMP e IPTC. O HEIF persistiu como `image/heif`. | Aprovado |
| CA3 / T09 · Promoção e referências (PRD F10, integração F11) | Somente publicar `available` com as duas saídas verificadas no mesmo commit. Preservar estado e referências privadas por arquivo. | A suíte conferiu `xmin` igual entre as duas variantes e o arquivo, mesma tentativa e perfil, associação ao arquivo correto, versões retornadas por S3, tamanho e SHA-256. A leitura anônima foi negada para as versões inspecionadas. Ao segurar só a segunda promoção do mesmo conjunto, PostgreSQL mostrou `available\|2` e `processing\|0` simultaneamente. Após liberar a promoção, o segundo arquivo ficou pronto e as referências do primeiro permaneceram idênticas. | Aprovado |
| CA4 / T10 · Queda, concorrência e reentrega (PRD F10) | Reconciliar a tentativa interrompida e processar o mesmo fato sem duplicar referências nem alterar o original e o recibo. | O runner viu `image.jpg` e `thumbnail.jpg` gravados no S3 enquanto o banco ainda mostrava `processing\|0`. Após derrubar e reiniciar as duas réplicas, republicou duas vezes o mesmo envelope no SNS e avançou o relógio além do lease. O banco registrou a tentativa 1 `failed` com `MEDIA_ATTEMPT_INTERRUPTED`, a tentativa 2 `succeeded`, uma deduplicação concluída e uma única dupla promovida. A versão, os bytes do original e o recibo foram conferidos. | Aprovado |
| CA4 / T10 · Falha definitiva (PRD F10, integração F13) | Marcar a mídia indisponível quando a falha é definitiva, conservar original e recibo e registrar o fato correlacionado. | Divergência controlada de checksum em PostgreSQL descartável levou a `prepare\|MEDIA_SOURCE_INTEGRITY\|failed\|0`. O runner encontrou um único `record.media-preparation.failed.v1` com correlação, causação, evento, registro e arquivo do fato de confirmação. A versão privada e o SHA-256 do original, além do recibo consultado e reenviado, permaneceram iguais. | Aprovado |

No HEIF suplementar, a fixture manteve a imagem HEVC sintética de 32 × 32 e usou `mif1` como marca principal, retirando `heic` do `ftyp`. SHA-256 da entrada `f2feca572484e43b8543d20f7544549044cb0234a19188c10c671afcd986894c`.

## Jornadas no navegador

| Jornada F08 | Chromium desktop | Android emulado | Efeito verificado |
| --- | --- | --- | --- |
| Selecionar foto, revisar, enviar por teclado e iniciar outro depoimento | Passou em 9,469 s | Passou em 9,221 s | Transferência e validação precederam o recibo. O recibo apresentou o instante em Brasília, com controles de conteúdo ausentes e prévia anterior revogada. A página passou na asserção axe e iniciou outro rascunho sem a foto anterior. |
| Capturar foto, recuperar envio interrompido e confirmar após o prazo de novos envios | Passou em 8,489 s | Passou em 5,787 s | O primeiro PUT foi interrompido pela fixture. A tentativa manteve a mesma chave, transferiu no segundo PUT e mostrou o recibo após o fechamento, com ações de novos envios indisponíveis. |

O Chromium headless 159.0.8154.6 executou as duas jornadas em cada perfil, com um worker e zero retentativas. Depois das jornadas, o runner encontrou dois arquivos confirmados por cenário, um de cada perfil, e verificou o fato de confirmação e as duas variantes prontas para todos os quatro arquivos. [Suíte Playwright](https://github.com/diorgenesbackes/scrapbook-tests/blob/a99d9b681788cd81f14e3c0427992f9febb86fa6/photo-submissions/photo-submission.spec.ts) · [JUnit sanitizado](e2e-junit.xml)

## Reprodução e alcance

Execute o comando em `scrapbook-tests` após instalar as dependências Playwright pelo lockfile e fixar os repositórios irmãos nos SHAs indicados.

```sh
DOCKER_CONTEXT=colima-scrapbook-f08-validation TMPDIR=/private/tmp python3 -B media_processing_stack.py --browser
```

O ensaio suplementar usou uma fixture de HEIF e isolou a promoção do segundo arquivo no PostgreSQL descartável. O ensaio de falha definitiva percorreu a rejeição imediata por integridade. O esgotamento das retentativas por erro transitório tem caso unitário no serviço. [Teste de integração](https://github.com/diorgenesbackes/scrapbook-tests/blob/a99d9b681788cd81f14e3c0427992f9febb86fa6/media_processing_stack.py) · [Testes do serviço](https://github.com/diorgenesbackes/scrapbook-service/blob/e9b6fe481d2b1cbd2a2e85fbe2ba6e60aa8dee9d/ScrapBook.UnitTests/Services/MediaPreparationServiceTests.cs)

O alcance local inclui JPEGs reais, câmera emulada e armazenamento e mensageria emulados. A F11 integra a galeria e a F13 encaminha o alerta. MediaConvert, áudio, vídeo, varredura histórica e reprocessamento operacional fazem parte das fases seguintes. As permissões efetivas de S3, KMS e SQS, dispositivos físicos, carga mista da F10 e aceite humano seguem para as etapas próprias de validação. [Aceite no PRD](https://github.com/diorgenesbackes/scrapbook-docs/blob/616cdc7359eec66f260040da78cdba2bb73368d3/PRD.md#f10-processamento-de-m%C3%ADdias-2) · [Plano da F10](../plan.md)
