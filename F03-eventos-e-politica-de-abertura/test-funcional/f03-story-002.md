# F03-STORY-002 · Aceite local

| Critério | Evidência |
| --- | --- |
| CA1 | O evento aceitou a criação imediata e uma troca de modo enquanto a janela estava flexível. Data civil inválida, passada ou antes do fim da janela foi recusada. |
| CA2 | O relógio injetado verificou os dois modos às 11:59:59, 12h e 13h de D+1 em Brasília. No banco real, a troca de imediato para futuro após o prazo gravou uma configuração travada. A próxima alteração foi recusada. |
| CA3 | Duas contas vinculadas concorreram com a mesma versão. Uma solicitação retornou `200`, a outra retornou `409 EVENT_VERSION_CONFLICT`. O navegador preservou o rascunho e exigiu novo envio após exibir a política atual. |
| CA4 | A consulta administrativa descartável confirmou autoria vencedora, correlação, versões, valores de antes e depois e retenção. A tentativa sem alteração conservou versão e trilha. Uma falha controlada de inserção da auditoria reverteu a atualização do evento. |

Os testes usaram HTTP real, PostgreSQL 18.6 e duas instâncias locais da API. [Spec](../spec.md#7-estratégia-de-testes) · [Revisão](../code-review/f03-story-002.md)
