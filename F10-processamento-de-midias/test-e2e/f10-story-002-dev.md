# E2E local da F10-STORY-002 em `dev`

A primeira execução integrada, em 27/11/2026, concluiu com código `0`. O Playwright registrou quatro jornadas aprovadas em 24,5 s, com zero falhas, erros, skips ou retries configurados. O ensaio completo levou 69,809 s. [Cartão](https://trello.com/c/I4MI0oAu) · [Spec](../spec.md) · [JUnit Playwright sanitizado](f10-story-002-junit.xml) · [Validação funcional](../test-funcional/2026-11-27_14-22-37/2-execucao.md)

| Repositório | SHA validado |
| --- | --- |
| `scrapbook-service` | `d7a6a4ba34cbf83ed6976ea2c55713c8f3fdef1e` |
| `scrapbook-infrastructure` | `b7bf2f746a2122b5511f1867bc2b9d2e384a3c58` |
| `scrapbook-tests` | `6338f1efec19c7ccbd5fe12f09e8ba7c224f9554` |
| `scrapbook-liquibase` | `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` |
| `scrapbook-ui` | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| `scrapbook-docs` | `d3c9700ddef5cf1ab4b02e287b24bc99a69f5510` |

O comando foi executado em `scrapbook-tests`, com `node_modules` reaproveitado após comparação byte a byte do lockfile. O laboratório usou Python 3.13.15, Node 26.13.0, npm 11.20.0, Playwright 1.65.0, Chromium 159.0.8154.6 e AWS CLI 2.37.10. A aplicação, Liquibase e PostgreSQL 18.6 rodaram no Compose com portas publicadas em loopback. S3 versionado, SNS e SQS usaram Moto com credenciais sintéticas. A imagem da API contém FFmpeg 9.1. O adaptador publicou um M4A gerado com FFmpeg 9.0.2 no host.

```sh
DOCKER_CONTEXT=colima-scrapbook-f10-validation TMPDIR=/private/tmp python3 -B audio_media_processing_stack.py --browser
```

| Jornada Playwright | Chromium desktop 320 px | Chromium Android · Pixel 7 emulado | Resultado |
| --- | --- | --- | --- |
| Selecionar WebM, revisar, identificar-se e enviar | 6,082 s | 5,882 s | A interface confirmou o recebimento no horário de Brasília. O recibo devolveu apenas estado e instante, ocultou o conteúdo e revogou a URL da prévia. A API respondeu com `Cache-Control: no-store`. |
| Retomar PUT de WAV após a interrupção | 5,764 s | 5,901 s | A interface anunciou a interrupção, leu o estado e retomou na mesma chave e caminho. A prova de recebimento às 13h foi aceita e o recibo durável chegou após o fechamento da janela. |

O runner consultou PostgreSQL após o Playwright. Os dois eventos de teste continham quatro arquivos confirmados e cada arquivo tinha uma única variante `playback` pronta e verificada. A etapa HTTP anterior cobriu cinco formatos, espera pelo provedor, resposta perdida com reentrega e saída corrompida. [Resultados da integração e observação de logs](f10-story-002-observacao.json)

A descoberta com `--list` encontrou os mesmos quatro casos com a configuração de URL de desenvolvimento. A configuração rejeitou URL ausente no modo `url` e origem remota no modo `local`. A execução em navegador ocorreu apenas no laboratório. Android foi emulado. A conversão real e o transporte EventBridge exigem a validação em homologação, assim como IAM e permissões. O player pertence à F11.
