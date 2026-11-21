# F03-STORY-001 · Validação funcional local

| Critério do card | Resultado observado |
| --- | --- |
| CA1 | Duas escritas concorrentes da mesma tentativa retornaram `201` e `200`, com um evento e um vínculo no slot 1. O evento apareceu na listagem e no detalhe com capa e mensagem padrão. |
| CA2 | Evento de 29/02 sugeriu 28/02 no ano seguinte. Uma data incompatível foi recusada com erro no campo. A UI aceitou uma data futura ajustada. |
| CA3 | Reenvio da mesma tentativa recuperou o evento. Entrada diferente com a mesma chave retornou `409 IDEMPOTENCY_KEY_REUSED`. |
| CA4 | A conta alheia recebeu lista vazia e o mesmo `404` de um evento inexistente. Identidade ausente foi recusada com `401`. A UI percorreu a paginação e o formulário pelo teclado. |

A validação usou duas contas sintéticas sobre PostgreSQL 18.6, API e UI em Docker Compose. A autenticação real da F02 e o aceite da homologação seguem como integrações posteriores. [Card da história](https://trello.com/c/6LlSdQxc) · [E2E](../test-e2e/f03-story-001.md)
