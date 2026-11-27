# E2E local da F10-STORY-003 em `dev`

Em 27/11/2026, o runner F08 → F10 concluiu em 100,399 s com código `0`. O Playwright registrou duas jornadas aprovadas em 12,402 s, com zero falhas, erros, skips ou retries configurados. A suíte HTTP e PostgreSQL verificou oito perfis de vídeo e três cenários de processamento assíncrono. [Cartão](https://trello.com/c/HesFEO9d) · [Spec](../spec.md) · [JUnit sanitizado](f10-story-003-junit.xml) · [Validação funcional](../test-funcional/2026-11-27_15-09-45/2-execucao.md)

| Repositório | SHA avaliado |
| --- | --- |
| `scrapbook-service` | `78abe427054ee420cab27da00b1a772534e390b9` |
| `scrapbook-infrastructure` | `1578ca0db337c58b0ce546e4ea252b89527443f1` |
| `scrapbook-tests` | `e19c13af9726e5605b3f7595e6a7d0637e11ecd1` |
| `scrapbook-docs` | `99044c55707eeab1f0539e5adab523d19e335734` |
| `scrapbook-ui` | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| `scrapbook-liquibase` | `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` |

O comando foi executado em `scrapbook-tests`, com `node_modules` reaproveitado após comparação byte a byte do lockfile. O laboratório usou Python 3.13.15, Node 26.13.0, npm 11.20.0, Playwright 1.65.0, Chromium headless 158.0.8127.18 e AWS CLI 2.37.10. A aplicação, Liquibase e PostgreSQL 18.6 usaram Compose com portas em loopback. S3 versionado, SNS e SQS usaram Moto com credenciais sintéticas. O FFmpeg 9.0.2 do host gerou as amostras e as referências. A imagem da API usa FFmpeg 9.1. A execução terminou com a mesma lista de contêineres ativos, redes e volumes que havia antes do teste.

```sh
DOCKER_CONTEXT=colima-scrapbook-f10-validation TMPDIR=/private/tmp python3 -B video_media_processing_stack.py --browser
```

| Caso de navegação | Projeto | Duração |
| --- | --- | --- |
| Selecionar, revisar e enviar MOV como depoimento | Chromium desktop, 320 × 740 px | 6,08 s |
| Selecionar, revisar e enviar WebM como momento | Chromium desktop, 320 × 740 px | 5,753 s |

Ambas as jornadas exibiram recibo genérico no horário de Brasília. O foco chegou ao título do recibo, a prévia foi retirada, a URL de prévia foi revogada e a consulta retornou apenas `state` e `confirmedAt`, com `Cache-Control: no-store`. O runner conferiu dois arquivos confirmados no PostgreSQL e verificou a variante `playback` de cada arquivo, vinculada à tentativa e à versão do objeto no S3 emulado.

A descoberta em modo `url` com `--list` encontrou os mesmos dois casos. A configuração rejeitou origem ausente no modo `url` e origem remota no modo `local`. A navegação foi executada somente no laboratório local. O adaptador validou os parâmetros do pedido de transcodificação e publicou um MP4 sintético correspondente. Os testes em ambiente remoto, com MediaConvert, EventBridge e IAM reais, ficam para homologação. Player e reprodução da variante pertencem à F11.
