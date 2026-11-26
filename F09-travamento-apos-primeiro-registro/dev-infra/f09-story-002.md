# F09-STORY-002 — Recuperação operacional de uma trava

O operador inspeciona a DLQ com uma identidade separada e registra a ocorrência em canal restrito. A retomada seleciona um único primeiro fato pelo `messageId` original. A aplicação revalida a fonte no PostgreSQL e devolve esse fato ao publicador normal. O SNS entrega a projeção canônica à fila da F09. [Contrato da F09](../spec.md#3-decisões-técnicas) · [Configuração do transporte](../../../scrapbook-infrastructure/README.md#travamento-após-primeiro-registro)

## Preparar a ocorrência

1. Identifique o ambiente, o candidato corretivo e a origem do alerta. Registre o incidente, a causa, a correção aplicada e o SHA completo do candidato na evidência de operação.
2. Confira o estado do fato em `outbox_messages`, da execução em `processed_messages` e do efeito no evento. O identificador usado é o UUID do envelope original. O identificador retornado pelo SQS identifica a entrega do transporte.
3. Quando houver uma entrega na DLQ, use a role associada a `DlqInspectionRoleName` e a URL `StructuralLockDeadLetterQueueUrl`. Confira os IDs IAM canônicos das duas roles ao configurar a stack. A política da role da tarefa bloqueia acesso à DLQ. Guarde o envelope em arquivo temporário de acesso restrito para reconciliar a origem. A mensagem permanece na DLQ durante o procedimento, até a retenção de 14 dias. O acesso concede consulta de atributos e recebimento da DLQ e preserva as restrições de envio à fila principal.
4. Confira `confirmedAt`. O replay ordinário admite 30 dias inclusive. Fora desse prazo, confronte fato, registro, trava e deduplicação e documente a decisão de reconciliação antes de autorizar a retomada. A F09 conserva a proteção de deduplicação por pelo menos 37 dias e enquanto houver trabalho recuperável.

O processo usa uma execução privada da mesma imagem da API, com conectividade ao banco da aplicação e a identidade operacional autorizada. O executável recebe uma referência de incidente e o SHA completo. Confira o SHA com o rótulo OCI `org.opencontainers.image.revision` da imagem e com a evidência da correção. Configure credenciais pelo mecanismo de segredos do ambiente. O comando termina após a operação e produz um código técnico de resultado.

## Retomada da Outbox

Com a causa da publicação corrigida, reabra o fato original em estado recuperável. O publicador existente reivindica uma nova lease e envia a mesma identidade lógica. Se a trava já estiver aplicada e for compatível com o primeiro fato, a republicação resolve a publicação falha e mantém a deduplicação concluída. Para retomada autorizada após os 30 dias, use `--reconcile-expired` com a evidência da reconciliação.

```sh
dotnet ScrapBook.Api.dll event-structural-lock-recover publish \
  "$f09_message_id" "$f09_incident_ref" "$f09_candidate_sha"
```

## Reconciliação e retomada da DLQ

Quando a entrega chegou à DLQ durante uma indisponibilidade do banco, use o envelope original para registrar o resultado terminal. O comando de reconciliação preserva a entrega e apenas registra a decisão durável.

```sh
dotnet ScrapBook.Api.dll event-structural-lock-recover reconcile-dlq \
  "$f09_message_id" "$f09_incident_ref" "$f09_candidate_sha" < "$f09_restricted_envelope"
```

Após corrigir a causa, autorize a retomada da execução em quarentena. O estado terminal anterior permanece disponível à F13. Para ocorrências reconciliadas e autorizadas após os 30 dias, acrescente `--reconcile-expired` conforme o registro do incidente.

```sh
dotnet ScrapBook.Api.dll event-structural-lock-recover consume \
  "$f09_message_id" "$f09_incident_ref" "$f09_candidate_sha" --rearm
```

O procedimento publica no SNS via Outbox para selecionar um único fato e reconstruir o envelope pela fonte persistida. O redrive nativo do SQS opera sobre o conteúdo da DLQ sem seleção individual. [Configuração de redrive no SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-dead-letter-queue-redrive.html)

## Conferir o resultado e a ativação

- Confira versão, correlação e resultado do comando. Após o reenvio, compare a identidade na Outbox e no consumidor, o resultado terminal existente e a aplicação registrada para F13. A auditoria do evento, a trava e a invalidação do convite devem representar um único efeito.
- Se a fonte durável não comprovar o primeiro fato, preserve o estado e registre a divergência. Confirmações históricas sem primeiro fato podem ser inventariadas com uma contagem agregada de eventos com registros confirmados e sem a entrada da F08. Decida cada reconciliação a partir da evidência autorizada da F08.
- Confirme os alarmes de idade da fila principal, mensagens visíveis da DLQ e falha terminal. Para esta última, configure `ServiceLogGroupName` com o grupo da API. `AlarmTopicArn` associa o canal operacional. Os alertas usam dimensões de fila ou fluxo. Identificadores técnicos ficam na evidência restrita. [Métricas do SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-available-cloudwatch-metrics.html)
- Para homologação, confira a role ECS, o isolamento da role operacional, as políticas SNS/SQS, o tópico de alerta e a disponibilidade das versões da F08 e F09. Depois de conferir o inventário inicial e as filas, habilite `EventStructuralLock__Enabled`. A desativação suspende a execução e mantém estados duráveis e pendências.
