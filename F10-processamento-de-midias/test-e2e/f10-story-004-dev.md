# E2E local da F10-STORY-004 em `dev`

As regressões F08 → F10 passaram nas dez jornadas de Chromium, sem falhas, erros ou skips. Os três runners concluíram com código `0`, verificando a confirmação pelo navegador e o processamento e a persistência de cada arquivo pelo serviço integrado. [Cartão](https://trello.com/c/dLVALXU9) · [Spec](../spec.md) · [JUnit sanitizado](f10-story-004-junit.xml) · [Validação funcional](../test-funcional/2026-11-28_13-52-21/2-execucao.md)

| Roteiro | Navegação | Asserções após o recibo | Tempo do runner |
| --- | --- | --- | --- |
| Foto | Seleção por teclado e captura emulada com retomada, em desktop e Android | Os quatro envios do navegador produziram duas variantes prontas cada e um fato por arquivo. O runner também verificou JPEG orientado, PNG transparente, HEIC, original preservado e retomada após vencer o lease das réplicas. | 132,606 s |
| Áudio | Seleção WebM e retomada do PUT, em desktop de 320 px e Android | Quatro arquivos ficaram disponíveis com M4A/AAC. Foram verificadas cinco famílias de entrada, resposta perdida após `CreateJob`, notificações antecipadas e duplicadas e saída inválida com fato F13. | 67,357 s |
| Vídeo | Seleção, revisão e confirmação de MOV e WebM, em desktop de 320 px | Dois arquivos ficaram disponíveis com variante MP4 H.264. O runner verificou oito perfis, resultado sem objeto, reentrega tardia, resposta perdida e saída corrompida com fato F13. | 76,028 s |

Os comandos foram executados em `scrapbook-tests` com `CI=true`, `DOCKER_CONTEXT=colima-scrapbook-f10-validation`, `TMPDIR=/private/tmp` e fuso `America/Sao_Paulo`.

```sh
python3 -B media_processing_stack.py --browser
python3 -B audio_media_processing_stack.py --browser
python3 -B video_media_processing_stack.py --browser
```

| Repositório | SHA avaliado |
| --- | --- |
| `scrapbook-service` | `3834529e86e1123a4c3119c67be5395478ea6612` |
| `scrapbook-infrastructure` | `8f6dd8cfb4226c467c48fc63e34667a82fb4b17a` |
| `scrapbook-tests` | `a62469c3711a27312c9e184ab72eedc91ff7b4ff` |
| `scrapbook-docs` | `c6eba10e48ece6d0b54b2635539ae3eef5591223` |
| `scrapbook-ui` | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| `scrapbook-liquibase` | `ac0e8517af47cdf4a61b81c715b0e8544040ecde` |

O laboratório usou Docker 29.5.2, Python 3.13.15, Node 26.13.0, npm 11.20.0, Playwright 1.65.0, FFmpeg 9.0.2 no host e AWS CLI 2.37.10. O lockfile dos testes foi comparado byte a byte antes de reutilizar as dependências. Cada runner subiu PostgreSQL 18.6, migrations Liquibase, API e UI em loopback e emulou S3 versionado, SNS e SQS com Moto. As quatro entradas de foto do JUnit foram reconstruídas do console do Playwright porque a execução seguinte reutilizou a mesma pasta de relatórios. Áudio e vídeo vêm dos arquivos JUnit originais, com saída e propriedades removidas. As referências de áudio e vídeo vieram da fixture local do contrato do MediaConvert. Conversão, eventos, permissões AWS e uso de câmera ou microfone físico ficam para os respectivos ensaios fora deste laboratório.
