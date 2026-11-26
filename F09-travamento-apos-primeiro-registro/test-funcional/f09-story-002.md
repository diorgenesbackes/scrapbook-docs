# F09-STORY-002 — Aceite funcional da retomada

Os ensaios usaram API `8bda7e9`, infraestrutura `b86dc79`, suítes `26ee890`, UI `57f4e63` e Liquibase `a9be70d`. As stacks prepararam PostgreSQL e SNS/SQS emulados em loopback, com eventos e identidades sintéticos. O ensaio HTTP integrado executou três réplicas do serviço. [Cenários da spec](../spec.md#7-estratégia-de-testes)

| Critério | Evidência |
| --- | --- |
| CA1 · Publicação sem marcação e ACK | Uma restrição temporária impediu marcar `published` depois da publicação. Após expirar a lease, uma fila observadora recebeu dois envelopes iguais. Uma falha controlada de ACK no emulador provocou reentrega após o commit. O efeito de negócio e o resultado F13 permaneceram únicos. |
| CA2 · Falha terminal de publicação e consumo | A ausência controlada do tópico produziu a falha `publish`. A restauração e a retomada preservaram sua evidência. Cinco tentativas de consumo recusadas antes do commit produziram `consume`, quarentena e entrega na DLQ. A fila principal drenou após a retomada. |
| CA3 · Reconciliação positiva | Em outra stack descartável, uma falha impediu gravar a marca terminal. A leitura do envelope legítimo na DLQ permitiu registrar o resultado com `reconcile-dlq`. A repetição preservou o resultado único. `consume --rearm` reenviou o mesmo fato. O recibo permaneceu acessível, a abertura foi preservada e o convite pendente foi invalidado uma vez. |
| CA3 · Histórico sem fato | O inventário encontrou um evento sintético com confirmação e sem fato verificável. O CLI recusou a retomada com código `3`. O recibo e a configuração permaneceram íntegros. O sinal agregado usou o ID fixo de catálogo `4202`. |
| CA4 · Sinais operacionais | Os logs registraram falha de ACK, falha terminal de publicação, retentativas de consumo, quarentena e amostras de backlog. Os sinais agregaram resultados e duração com campos técnicos limitados. |

Na primeira rodada da reconciliação positiva, uma segunda entrega da DLQ emulada permaneceu indisponível para leitura durante a inspeção. A repetição isolou esse caso em uma DLQ exclusiva e confirmou o envelope e a reconciliação. Os controles operacionais reais seguem para o aceite em homologação. [Procedimento de retomada](../dev-infra/f09-story-002.md)
