# Verificação defensiva da F10-STORY-005 em dev

28/11/2026. Controles de privacidade, escopo de permissões e configuração de ativação vinculados a CA3, CA4 e T10 do [cartão F10-STORY-005](https://trello.com/c/VIdGX9ti). A revisão considerou os requisitos de F10 no PRD, a especificação e o procedimento de ativação e capacidade.

| Repositório | SHA integrado verificado |
| --- | --- |
| Service | `d7ac67dc9095ea55c1a4c402e91e9b5a2568e43c` |
| Tests | `c1782eb0636d7038680c59654ea361c60299fe82` |
| Infraestrutura | `0ec49e6b121078d427bfe2c26829fe36346ff0a7` |
| Docs | `bd724ecb50b9f0e62e81a629a81a252a3e032274` |
| UI | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| Liquibase | `ac0e8517af47cdf4a61b81c715b0e8544040ecde` |

## Resultado

Nenhum achado de segurança confirmado no alcance local verificado. T10 tem cobertura local para políticas declaradas, privacidade das dimensões, leituras anônimas e recibo sintético. A identidade efetiva de homologação e a entrega e visibilidade efetivas dos logs, métricas e alarmes na AWS permanecem pendentes.

| Método | Controle e prova | Resultado |
| --- | --- | --- |
| White box | O bucket da F08 declara versionamento, KMS, os quatro bloqueios de acesso público, ownership do bucket e política TLS. A role de execução do MediaConvert tem trust restrito ao serviço, conta e fila e usa apenas staging e saída. A role de runtime usa a fila e a role de execução indicadas, e as políticas de entrega e redrive limitam filas, origens e operadores. Conferidos os templates `photo-submissions`, `media-processing`, `media-transcoding` e a suite de IaC. | Aprovado no alcance da configuração versionada. Os 28 testes de infraestrutura passaram, incluindo 15 de mídia. |
| White box | `MediaProcessingTelemetry` normaliza tipo por lista permitida. Stage e outcome são enums, coorte é controlada, o backlog consulta uma lista fixa de tipos e os filtros da F10 usam etapa, tipo, resultado e coorte. Os números de série, chaves, versões de artefato, endereços assinados e identificadores de negócio ficam fora das dimensões e mensagens de agregado revisadas. O `eventId` fixo 4301 identifica o evento do logger. | Aprovado nas fontes, no `MeterListener` do unitário e na renderização isolada de dois intervalos pela classe de monitoramento e pelo formatter reais. O marcador sintético de referência privada virou `unknown` no agregado e ficou ausente do JSON, incluindo o campo `message`. |
| White box | Defaults da API mantêm `MediaProcessing:Enabled`, `MediaAudioProcessing:Enabled` e `MediaAudioProcessing:VideoEnabled` em `false`. `MediaProcessing:Paused` começa em `false` e é uma opção separada após habilitação. A pausa impede workers de publicação, admissão e foto, preserva monitoramento e limita áudio/vídeo à reconciliação de tentativas com intenção de submissão. | Aprovado no código e nos testes unitários. O runner da própria API devolveu código de saída 3 para recuperação desabilitada e para `history` e `redrive` com `--apply` durante a pausa. Os três caminhos foram avaliados antes de conexão com banco. |
| Grey box, evidência compartilhada da execução E2E | Na stack descartável, a leitura autorizada dos originais e das variantes de foto pelas versões previstas conferiu bytes e checksum, e as versões derivadas conferiram o vínculo de arquivo e tentativa. A foto de teste com orientação e GPS gerou variantes com dimensões esperadas e sem os segmentos examinados. O recibo legítimo preservou `state`, `confirmedAt` e `Cache-Control: no-store`. | Aprovado para as fixtures de foto da suíte integrada. |
| Black box guiado, evidência compartilhada da execução E2E | GET sem assinatura às mesmas versões cuja existência e leitura autenticada foram comprovadas no S3 emulado. O teste exige 403 ou 404 e falharia diante de uma leitura permitida. A fonte da suíte foi lida antes de analisar a evidência externa. | Aprovado para a amostra sintética de originais e variantes de foto. A evidência de execução confirma o conjunto de status aceito pela asserção. |
| Grey e black box em homologação | Permissões efetivas, acesso anônimo e por escopo no S3 real, IAM e KMS do runtime, proteção de logs e alarmes no ambiente implantado e referências e artefatos privados de áudio/vídeo produzidos pelo MediaConvert real. | Bloqueado nesta rodada por falta de ambiente implantado e identidade AWS no escopo deste ensaio. |

## Evidências

- `python3 -B -m unittest discover -s tests -v` no repositório de infraestrutura integrado executou 28 casos, exit 0. O recorte `test_media*.py` executou 15 casos, exit 0. Python 3.13.15.
- O VSTest 18.7 executou 471 casos com `FullyQualifiedName~Media`, exit 0. A execução usou os binários já compilados do candidato `062975160c99e9a33e0a1a21e00b60e33c3f56a1`, cujo tree `1f7215b6774186679e366aab7afdb0f95256f2b7` é idêntico ao tree do merge integrado. O runner exigiu o acesso ao socket de loopback interno. O restore iniciado neste checkout ficou sem conclusão e a recompilação local independente ficou pendente. SDK .NET 10.0.401.
- O ensaio isolado de logs usou as assemblies do mesmo tree com a instrumentação e o formatter reais. Duas entradas foram conferidas campo a campo em memória e nenhum texto privado foi exportado. O ensaio de CLI usou as mesmas assemblies e um ambiente local mínimo.
- A execução E2E integrada `python3 -B media_processing_stack.py --browser` passou com quatro jornadas Chromium e as asserções HTTP, PostgreSQL e S3 da suíte, exit 0. A leitura autorizada e a negação anônima constam de `read_object`, `assert_private`, `assert_original` e `assert_ready`. O log compartilhado da execução é `e2e-chromium.log`.
- O primeiro ensaio integrado de `media_pause_stack.py` obteve variantes prontas após retomar e falhou ao observar o estado publicado da Outbox na janela do teste. Com o ajuste do relógio da fixture, o reteste local passou e preservou recibos e originais privados, com duas fotos disponíveis, quatro variantes prontas e dois fatos publicados. [Evidência E2E](../test-e2e/f10-story-005-dev.md)
- O probe HTTP adicional de comparação entre dois bearers e análise de marcadores nos logs da stack foi preparado em área privada. A rodada utilizou a prova de objetos da E2E integrada e a renderização isolada para preservar a janela da medição de carga. O probe adicional ficou sem execução.

Os arquivos locais de execução são `security-infra-all.log`, `security-templates.log`, `security-dotnet-tree-test.log`, `security-structured-log.log` e `security-cli-observations.json`. Verificação sem alterações nos repositórios.
