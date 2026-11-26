# F09-STORY-001 — Transporte do primeiro registro

O tópico SNS recebe os eventos internos. A assinatura seleciona `event.first-record-confirmed.v1` no atributo `type` e entrega o envelope diretamente à fila Standard da F09. A fila aceita o tópico configurado, e a role da tarefa tem publicação no tópico e consumo na fila própria. A DLQ preserva entregas que esgotaram as tentativas. [Template](../../../scrapbook-infrastructure/config/messaging/event-structural-lock.template.json) · [Configuração da aplicação](../spec.md#parâmetros-operacionais-propostos)

O ensaio local criou recursos SNS e SQS descartáveis no Moto e confirmou que um fato de mídia ficou fora da fila da F09 e o fato de primeira confirmação chegou com o corpo JSON original. O candidato ainda depende de configurar a fila e o tópico no ambiente publicado. O aceite em homologação confere a entrega nativa e as permissões IAM. [Estratégia de testes](../spec.md#7-estratégia-de-testes)
