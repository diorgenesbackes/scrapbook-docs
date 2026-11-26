# F09-STORY-002 — Resultados duráveis e retomada

O publicador mantém o identificador lógico durante retentativas e condiciona a conclusão à lease vigente. Uma falha definitiva verificável registra o estado da Outbox e o resultado `publish` na mesma transação. O consumidor deixa o SQS reentregar quando a gravação falha, confirma a entrega depois do commit e distingue falha de ACK de falha de processamento. No último recebimento, uma origem confirmada pode gravar quarentena e resultado `consume` em uma nova transação. [Spec da F09](../spec.md#3-decisões-técnicas)

## Outbox da F13

| Tipo | Campos do payload | Persistência |
| --- | --- | --- |
| `event.structural-lock.applied.v1` | `eventId`, `confirmedAt`, `lockedAt` | A transação que aplica a trava também grava a invalidação aplicável, auditoria, deduplicação e o resultado. |
| `event.structural-lock.failed.v1` | `eventId`, `confirmedAt`, `failedAt`, `stage`, `code` | Um resultado por primeiro fato e etapa (`publish` ou `consume`). Uma retomada posterior preserva o histórico terminal. |

Os resultados usam UUID próprio e determinístico, `causation_id` do primeiro fato, versão de protocolo `1`, correlação validada e instantes em UTC. A publicação ativa seleciona apenas `event.first-record-confirmed.v1`. Os resultados permanecem pendentes na Outbox para a futura integração da F13. [Primeiro fato](../spec.md#mensagem-de-entrada) · [Resultados para F13](../spec.md#resultados-para-f13)

## Retomada e histórico

O comando operacional da mesma imagem coordena a verificação da origem, a reconciliação com o estado persistido, a auditoria e a reabertura do fato original. A deduplicação permanece armazenada. Reabrir uma execução em quarentena requer a autorização explícita do operador. O prazo ordinário inclui o instante de 30 dias após a primeira confirmação. Uma retomada fora do horizonte exige a opção de reconciliação e a conferência de registro, trava e deduplicação. [Procedimento](../dev-infra/f09-story-002.md)

O processamento usa os grants e as tabelas já aplicados. O sucesso mantém as permissões da F03, a decisão legítima da F04 e os contratos HTTP existentes. A monitoração amostra o backlog pendente e falho e a idade do trabalho. Cada réplica observa o mesmo backlog global e a agregação usa o máximo ou uma amostra por período. A cada minuto por réplica, `event.structural_lock.processing_interval` registra contagens locais de publicação, consumo e reentrega, amostras de duração até o commit, média, máximo e quantidade acima de 5 s. O sinal usa códigos e contagens com cardinalidade limitada. A coleta de métricas, os alarmes e as permissões efetivas seguem para o aceite no ambiente publicado.
