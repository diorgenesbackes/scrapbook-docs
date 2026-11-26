# F09-STORY-002 — Verificação defensiva

A revisão independente conferiu as árvores integradas `8bda7e9` (API), `b86dc79` (infraestrutura) e `26ee890` (suítes). A validação HTTP usou identidades e eventos sintéticos em uma stack descartável de loopback. [Critérios](../spec.md#7-estratégia-de-testes)

| Controle | Evidência |
| --- | --- |
| Autorização e privacidade | Identidades vinculadas mantiveram as ações permitidas. Consultas anônimas e de fora do vínculo foram recusadas. Cache, correlação e contratos do recibo e do convite passaram nas asserções de minimização. |
| Origem e horizonte | Origem incompatível ficou sem efeito nos testes unitários. A reconciliação integrada conferiu o envelope da DLQ contra Outbox, registro e evento. A retomada manteve o `messageId`, o resultado terminal e a deduplicação. Os unitários cobriram +30 dias inclusive e a condição de reconciliação posterior. |
| Permissões de retomada | O comando funciona como execução operacional da imagem. A role da tarefa recebe e exclui mensagens na fila principal e a política bloqueia acesso à DLQ. A role separada inspeciona a DLQ, e o SNS mantém a entrega canônica à fila principal. Quatro testes do template passaram. |
| Diagnósticos e auditoria | Os logs das três réplicas passaram na verificação dos marcadores sintéticos. Falhas de `publish` e `consume` e amostras intervalares usaram códigos, contagens e durações limitados. A identidade DML preservou a auditoria de acréscimo. |

A revisão levou o adaptador da CLI a sanitizar também recusas do provedor de dados e retirou `ChangeMessageVisibility` da role da tarefa. A visibilidade e a reentrega seguem a configuração da fila. [Revisão técnica](../code-review/f09-story-002.md)

O reteste do CLI com uma credencial sintética recusada terminou com código `4` e `RECOVERY_UNAVAILABLE`, sem saída padrão, detalhe de banco ou SQLSTATE. A imagem emitiu também um aviso nativo pela ausência da biblioteca GSSAPI. A verificação separou esse aviso do resultado técnico e confirmou a ausência dos marcadores de credencial.

Grants IAM, configuração TLS, entrega SNS/SQS e alvo dos alarmes seguem para homologação. [Configuração operacional](../dev-infra/f09-story-002.md)
