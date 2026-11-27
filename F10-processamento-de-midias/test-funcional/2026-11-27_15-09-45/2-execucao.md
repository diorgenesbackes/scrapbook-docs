# Validação funcional local da F10-STORY-003 em `dev`

Em 27/11/2026, a primeira rodada integrada passou nos oito perfis de vídeo, nos três cenários assíncronos e nas duas jornadas Playwright. A cobertura local dos critérios está descrita abaixo. O aceite da matriz completa de codecs e da saída real do MediaConvert segue pendente em homologação. [Plano do ensaio](1-plano.md) · [Comando, ambiente, SHAs e JUnit](../../test-e2e/f10-story-003-dev.md) · [Critérios no cartão](https://trello.com/c/HesFEO9d)

| Critério | Resultado observado | Estado |
| --- | --- | --- |
| CA1 · F08 → F10, FR-020 | As amostras confirmadas de MP4, MOV e WebM produziram uma variante privada `playback` com `video/mp4` e H.264. As amostras com áudio tiveram AAC mono e as fontes mudas permaneceram sem faixa. O adaptador recebeu os formatos de contêiner esperados e observou a versão e o SHA-256 da entrada privada da tentativa. PostgreSQL vinculou o ID do job, a versão e o SHA-256 da saída correspondentes. O runner conferiu também a versão, os bytes e a privacidade de cada original. | Aprovado para as amostras do laboratório. Matriz completa de codecs e conversão real na AWS pendentes. |
| CA2 · FR-020 | O contrato enviado ao adaptador especificou `FIT_NO_UPSCALE`, pixel quadrado, rotação e taxa de saída explícita com `DUPLICATE_DROP`. As propriedades das variantes verificadas e gravadas pela aplicação coincidiram com a matriz abaixo. | Aprovado para configuração e validação local. Orientação nos píxeis, ausência de corte ou bordas e comportamento real de VFR do MediaConvert pendentes em homologação. |
| CA3 · FR-020 | Com `COMPLETE` e objeto ausente, o arquivo continuou em `processing` sem versão vinculada e registrou `MEDIA_VARIANT_STORAGE_UNAVAILABLE`. Após gravar o MP4 íntegro, a mesma tentativa tornou o arquivo disponível. A perda da resposta após criação seguida de resultado antecipado convergiu em uma tentativa. A reentrega após prontidão preservou todo o estado consultado, inclusive versão e SHA-256. | Aprovado no laboratório para saída ausente, resultado antecipado e reentrega tardia da mesma tentativa. Para tentativa divergente, ver cobertura complementar abaixo. |
| CA4 · F10 → F13, FR-020 | Um resultado `COMPLETE` cujo objeto continha bytes inválidos levou arquivo, trabalho, tentativa e variante a `failed`, sem versão promovida. A Outbox registrou exatamente um fato `record.media-preparation.failed.v1` com etapa `verify` e correlação e causação ligadas ao fato de confirmação. A leitura versionada comprovou original privado intacto e a consulta e o replay preservaram o recibo. | Aprovado no laboratório. O encaminhamento do alerta pertence à F13. |
| Regressão · F08-STORY-004 | O convidado selecionou, revisou e enviou um MOV em depoimento e um WebM em momento. As duas jornadas chegaram ao recibo no horário de Brasília. A consulta devolveu o estado com `no-store` e a interface retirou a prévia. O runner verificou em PostgreSQL um arquivo confirmado e pronto para cada jornada. | Aprovado em Chromium desktop. |

## Perfis verificados

| Fonte sintética | Propriedades da variante verificada |
| --- | --- |
| MP4/H.264 com AAC mono, paisagem de 320 × 180 a 25 fps | 320 × 180 a 25 fps com AAC mono |
| MOV/H.264 mudo, raster de 320 × 180 a 24 fps e rotação de 90° | Retrato de 180 × 320 a 24 fps, mudo |
| WebM/VP9 com Opus mono, quadrado de 240 × 240 a 60 fps | 240 × 240 a 30 fps com AAC mono |
| MP4/H.264 mudo, anamórfico de 320 × 240 a 24 fps e SAR 4:3 | 320 × 180 a 24 fps com pixel quadrado, mudo |
| MP4/H.264 mudo, VFR de 256 × 144, com intervalos de 1/12 s e 1/24 s | 256 × 144 a 12 fps, mudo |
| MP4/H.264 mudo de 2.048 × 1.152 a 24 fps | 1.920 × 1.080 a 24 fps, mudo |
| MP4/H.264 com duas faixas AAC mono de 3 s e 5 s, vídeo de 176 × 144 a 24 fps | 176 × 144 a 24 fps com uma faixa AAC mono de 3 s na referência |
| MOV/H.264 mudo 4:4:4, raster de 352 × 288 a 24 fps e rotação de 90° | Retrato de 288 × 352 a 24 fps, mudo |

O `ffprobe` caracterizou as amostras e referências sintéticas. O laboratório exigiu que a aplicação verificasse os bytes do artefato e salvasse suas propriedades antes de disponibilizar a variante. O teste integrado com duas faixas verificou o pedido e a promoção da referência da faixa principal. A verificação dos bytes do estágio e da normalização 4:4:4 pertence também aos casos `StagesThePrimaryAudioWithoutInheritingTheLongerSecondaryTrackAsync` e `StagesRotatedH264InProgressiveYuv420WithoutChangingTheSourceGeometryOrCadenceAsync` em `S3MediaVideoStorageTests`.

## Cobertura complementar

O runner integrado repetiu a notificação tardia da tentativa que produziu a variante pronta. A fronteira de duas tentativas para o mesmo arquivo foi validada em `MediaVideoPreparationServiceTests.IgnoresThePriorVideoCallbackWhileTheReplacementAttemptRemainsActiveAsync`. O unitário rejeitou o resultado da tentativa anterior e promoveu somente o objeto da substituta. A mesma suíte verificou a rejeição de uma conclusão com referência de outra tentativa ou arquivo e a falta de promoção quando o lease é perdido após a leitura. `S3MediaVideoStorageTests.RejectsMissingOrChangedPlaybackVersionsAsync` verificou a divergência de versão na leitura. Os unitários passaram no candidato aprovado, com conteúdo idêntico ao do merge validado aqui. [Revisão técnica](../../code-review/f10-story-003.md)

Os itens T01 e T09 do cartão exigem a matriz e a integração real em homologação, incluindo MediaConvert, EventBridge, SQS, S3 e IAM. A métrica de prontidão sob concorrência tem evidência de performance própria. A interface de reprodução pertence à F11. [Aceite da F10 no PRD](../../../PRD.md#f10-processamento-de-mídias-2) · [Estratégia de testes da spec](../../spec.md#7-estratégia-de-testes)
