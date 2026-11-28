# F10-STORY-005 — Ativação e capacidade em homologação

A ativação em homologação depende do candidato integrado de F10-STORY-001 a F10-STORY-004, do catálogo `f10-story-004-media-recovery` e do fluxo de confirmação da F08. Registre o ambiente, os SHAs, as versões da API e do Liquibase e o resultado dos respectivos workflows antes do ensaio. [Cartão](https://trello.com/c/VIdGX9ti) · [Planejamento](../plan.md) · [Entrega de recuperação](../delivery/f10-story-004.md)

## Preparação

1. Confira a execução de publicação da API e do Liquibase pelo SHA em `hlg`. Se a reserva de versão falhar, use o log da etapa para localizar o erro e verifique o valor de `RELEASE_SERIES` no repositório e a permissão de criar a tag de release. Essas verificações são independentes do sucesso dos checks do PR.
2. Confira a geração ativa confirmada pelo Liquibase no ambiente. O workflow da API usa `DATABASE_GENERATION` para verificar esse marcador antes da ativação no ECS. Preserve a ordem banco → API e registre também `aws ecs describe-services` e a saúde pelos endpoints publicados.
3. Confira conta, região e quota autorizadas, fila MediaConvert, role de execução, bucket da F08 versionado com KMS, SNS/SQS de confirmação, EventBridge/SQS de resultado e as três DLQs. Registre a concorrência da fila e a quota regional compartilhada com as demais filas. Valide o acesso negado a uma origem ou chave de outro escopo com objetos sintéticos.
4. Provisione o [template de observabilidade](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/hlg/config/media/media-observability.template.json) sobre o grupo Standard de logs JSON já existente. Use um nome de monitoramento único no ambiente e os nomes das três filas de falhas existentes. O ARN SNS de operação é opcional. Quando configurado, confirme a entrega de um alarme com a assinatura operacional. [Filtros do CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)
5. Prepare dois eventos isolados e vazios em homologação para a massa sintética. Use identidades autorizadas e endpoints do ambiente, com vínculo e aviso da F06. Confira a fonte dos timestamps de commit com a identidade de leitura do ensaio. Registre elegibilidade e configuração antes de liberar tráfego.

## Perfil e medição

| Por evento | Registros | Arquivos |
| --- | ---: | ---: |
| Depoimento em texto | 25 | 0 |
| Foto individual | 25 | 25 |
| Momento de duas fotos | 10 | 20 |
| Áudio | 20 | 20 |
| Vídeo | 20 | 20 |
| Total | 100 | 85 |

Libere os dois grupos juntos e registre a concorrência efetivamente atingida, os tipos de entrada, a distribuição das amostras, o pico de trabalhos no banco e a concorrência da fila. Confira p50, p95 e p99 da confirmação da F08 e da prontidão F10 por tipo e total. A janela individual de prontidão mede `confirmedAt` até o commit em que o conjunto exigido do perfil ativo ficou disponível. Registros de texto compõem o tráfego da F08. O denominador da F10 é formado por todos os arquivos confirmados, inclusive os que falharam ou ficaram pendentes. Em 170 confirmações, o mínimo para a meta é 162 prontas em até **300 s**. [Critérios da F10](../spec.md#aceite-rastreado-ao-prd)

Para evidência precisa no PostgreSQL, confirme `track_commit_timestamp` e resolva `pg_xact_commit_timestamp` do efeito comprometido. Confira se as variantes obrigatórias e a mudança do arquivo para `available` estão na mesma transação do perfil ativo. Guarde o instante de confirmação do fato canônico ou do trabalho e a versão de artefato verificada. Trate timestamp de commit ausente, coorte incompleta ou janela truncada como resultado inconclusivo. `updated_at` é um marcador da aplicação gravado antes de concluir a transação. Exporte só identificador sintético, coorte, tipo, estado e instantes para análise e mantenha a origem bruta em área privada. A configuração da base de homologação fica com seu responsável.

Na rodada de borda, cubra os limites F08 de 20 MB para fotos, 50 MB para áudio e 500 MB para vídeo e as durações máximas de 300 s para áudio e vídeo de depoimento e 60 s para vídeo de momento. Registre próximo do limite e exatamente no limite onde o contêiner permitir. Inclua HEIC, foto orientada e com localização sintética, JPEG pequeno, WebM de áudio, MP3, WAV, M4A, Ogg, MP4, MOV e WebM de vídeo, saídas quadradas, verticais, anamórficas, VFR e material com cadência acima de 30 fps. Confira a compatibilidade no provedor real e inspeção de codec, contêiner, duração, faixa de áudio, proporção, rotação, fps, dimensões e ausência de localização nas variantes. [Limites do envio F08](../../F08-envio-e-confirmacao/spec.md) · [Perfis da F10](../spec.md#3-decisões-técnicas)

Observe CPU, memória, espaço temporário, conexões do banco, fila de conversão e erros de S3/MediaConvert. O parâmetro `ConcurrentJobs` da stack de conversão limita a fila. Confira sua relação com a quota regional e a capacidade das réplicas de preparar entrada e verificar as saídas. Cada réplica usa até dois slots de foto e dois de áudio/vídeo com as flags atuais. Uma vaga por tipo atende confirmações novas. A outra alterna a preferência, conforme a política da F10-STORY-004. Meça histórico separadamente a partir da admissão, usando o corte de ativação fixo e lotes auditados de até 100 por evento. [Quota e filas MediaConvert](https://docs.aws.amazon.com/mediaconvert/latest/ug/working-with-on-demand-queues.html) · [Recuperação operacional](../dev-back/f10-story-004.md)

## Sinais de operação

| Sinal | Fonte e leitura |
| --- | --- |
| Backlog e consumo de confirmação e resultado | `AWS/SQS` por `QueueName`, com `ApproximateNumberOfMessagesVisible`, `ApproximateNumberOfMessagesNotVisible`, `ApproximateAgeOfOldestMessage`, `NumberOfMessagesReceived` e `NumberOfMessagesDeleted`. São medidas operacionais do transporte. |
| Mensagens retidas em DLQ | Alarme nativo para mensagens visíveis da confirmação, consumo de resultado e falha de entrega EventBridge. |
| Publicação, consumo, tentativa, falha e reconciliação | `ProcessingSamples` por `Stage`, `MediaType` e `Outcome` no namespace `ScrapBook/Media/<MonitoringName>`. Agregue contagens entre réplicas por soma e revise a janela de intervenção. |
| Latência dos estágios | `ProcessingDurationSumSeconds` dividido por `ProcessingDurationSamples` por etapa, tipo e coorte. Preserve os histogramas cumulativos no log para leitura dos intervalos. |
| Intervalos de prontidão | `ReadySamples`, `ReadyWithinTarget` e `ReadyDurationSumSeconds` por tipo/coorte, agregados por soma. Use máximo em `ReadyMaximumSeconds`. Os intervalos do serviço observam commits concluídos. O ensaio de aceite usa a coorte completa e os tempos por arquivo. |
| Backlog persistido da Outbox de mídias | `OutboxPending`, `OutboxFailed` e `OutboxOldestAgeSeconds`. Use máximo no período para que amostras do mesmo estado entre réplicas não somem o estoque. |
| Trabalhos persistidos | `JobsPending`, `JobsActive`, `JobsReconciling` e `JobsOldestAgeSeconds`. Aplique máximo nas amostras por tipo/coorte e use os estados para confirmar a drenagem em pausa. |
| Uso do provedor por fila | `JobsSubmittedCount`, `JobsCompletedCount`, `JobsErroredCount`, `StandbyTime` e `TranscodingTime`, conforme publicados pelo MediaConvert. Correlacione os sinais de fim do job com a fila de resultados e a taxa de verificação. |

Campos técnicos de métricas limitam-se a etapa, resultado, tipo e coorte. No painel e nos relatórios compartilhados, use agregados por janela. Restrinja evidências com correlação de problema à operação autorizada. [Métricas SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-available-cloudwatch-metrics.html) · [Métricas MediaConvert](https://docs.aws.amazon.com/mediaconvert/latest/ug/metrics.html)

## Ativação, pausa e recuperação

1. Publique a API com as flags de processamento no estado seguro previsto. Confira saúde, migração, métricas, quotas, destinos e versão dos originais com poucos envios sintéticos. Registre um único corte UTC fixo em `MediaRecovery__HistoryBeforeUtc` para o serviço e o runner. Confira a publicação independente da F09 e a confirmação da F08.
2. Habilite a preparação de fotos e acompanhe recibos, filas e variantes. Em seguida habilite áudio e só então vídeo com os formatos suportados no ambiente. Confira uma variante de cada perfil e a saúde de cada réplica antes de ampliar. Use `MediaProcessing__Enabled`, `MediaAudioProcessing__Enabled` e `MediaAudioProcessing__VideoEnabled` conforme a configuração da F08. [Configuração de mídia](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/hlg/README.md)
3. Rode o perfil misto e a rodada de borda com a capacidade documentada. Compare os picos observados com os limites da fila e do ambiente antes de ajustar a capacidade. Em caso de erro que afete a prontidão ou o recibo da F08, pause a aquisição e investigue os sinais da mesma coorte.
4. Para pausar, publique a configuração com `MediaProcessing__Paused=true` e confirme que todas as réplicas convergiram. Suspenda aplicações manuais de histórico/redrive durante a troca e use a mesma opção de pausa no runner. Observe o fim do trabalho já em curso, a expiração de leases antigos e a reconciliação das tentativas de áudio/vídeo que já possuíam intenção de submissão. Preserve a Outbox, as filas, o bucket, os recibos, os originais e as referências prontas. Registre trabalhos ambíguos que exigem intervenção.
5. Para retomar, publique `MediaProcessing__Paused=false`, confirme novas aquisições e monitore a drenagem da fila e os percentis por tipo. Mantenha o corte fixo e a mesma política de deduplicação. Faça recuperação histórica em lotes após validar a capacidade de receber envios novos. Para restaurar a versão anterior da API, mantenha as flags de processamento desabilitadas durante a troca se a versão anterior não reconhecer a opção de pausa. Preserve o catálogo aditivo e os dados já processados. Use o runbook de release em caso de recuperação do artefato.

Registre horário da mudança, configuração observada, sinais, efeitos sobre recibos e estado das variantes, revisão aprovada, resultado da retomada e recursos pendentes. Os resultados locais com emuladores evidenciam o fluxo técnico. O aceite com S3, MediaConvert, EventBridge e SQS reais pertence à homologação. [Validação prevista](../spec.md#7-estratégia-de-testes)
