# F09-STORY-002 — Revisão técnica

A revisão independente percorreu a publicação condicional à lease, a unicidade dos resultados da F13, as transações de consumo e recuperação, o ACK do SQS, o horizonte de replay e as políticas operacionais. [Escopo](../spec.md#3-decisões-técnicas)

| Achado inicial | Correção revisada |
| --- | --- |
| P2 · Um evento já travado podia manter a publicação original em `failed` após uma resposta perdida do SNS | A retomada de publicação reenfileira o primeiro fato quando a trava persistida é compatível. A deduplicação concluída e o resultado terminal permanecem gravados. |
| P2 · O histograma usava um instante anterior ao commit | A duração agora inclui o commit. O teste avança o relógio durante a transação e verifica a duração registrada. |
| P2 · As métricas internas dependiam de um coletor ainda ausente no ECS | O worker passou a registrar também totais intervalares sanitizados, incluindo duração e reentregas. |
| P2 · Nomes IAM com diferença de caixa podiam contornar a comparação literal | A política da role da tarefa bloqueia explicitamente o acesso à DLQ. O procedimento de homologação confere os IDs IAM das identidades. |
| P2 · As asserções iniciais observavam reaquisição da lease e contagem da DLQ | O ensaio captura os dois envelopes publicados e confere igualdade. A leitura no laboratório confere a identidade lógica da entrega. |

Uma sonda isolada observou cinco recebimentos na origem e o envelope canônico na DLQ. No laboratório integrado, a inspeção inicial em lote expirou após o encaminhamento. O runner passou a receber uma mensagem por vez, com visibilidade de 1 s na inspeção, sem alterar a política da fila em homologação.

O build do serviço terminou com zero erros e avisos. No ambiente local, os 356 testes da API e os 13 da infraestrutura passaram. O CLI confirmou as respostas sanitizadas para entrada inválida e ausência de configuração. A descoberta Playwright listou três cenários da F09.
