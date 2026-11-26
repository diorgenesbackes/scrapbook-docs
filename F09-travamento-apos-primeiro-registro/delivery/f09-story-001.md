# F09-STORY-001 — Entrega da trava estrutural

A primeira confirmação da F08 aciona a trava das informações estruturais, encerra o convite pendente e mantém as regras de abertura e os responsáveis já vinculados. A interface reconsulta o estado e retira as ações de edição e convite depois da trava. [Card](https://trello.com/c/cWad9rrV) · [Especificação](../spec.md)

## Integração em `dev`

| Componente | PR | SHA do merge |
| --- | --- | --- |
| API | [#12](https://github.com/diorgenesbackes/scrapbook-service/pull/12) | `4f3f0249ed54c73706526ac74d70372352427801` |
| Interface | [#9](https://github.com/diorgenesbackes/scrapbook-ui/pull/9) | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| Infraestrutura | [#9](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/9) | `56759f34ce0fe8b8103a6f0e81f57e0d53929862` |
| Suítes | [#10](https://github.com/diorgenesbackes/scrapbook-tests/pull/10) | `7138d3d4e8d096c489f57643f9267a87940ffaa9` |
| Documentação inicial | [#17](https://github.com/diorgenesbackes/scrapbook-docs/pull/17) | `99dfdbaf0729d67fc8d04857d9227a08bd4815e6` |
| API, correção de correlação | [#13](https://github.com/diorgenesbackes/scrapbook-service/pull/13) | `1ee9d816b2339ed4fff79f6d209462f696e03fbe` |

## Evidências do candidato

- [Revisão técnica](../code-review/f09-story-001.md)
- [E2E e integração HTTP](../test-e2e/f09-story-001.md)
- [Aceite funcional](../test-funcional/f09-story-001.md)
- [Carga](../test-performance/f09-story-001.md)
- [Verificação defensiva](../test-security/f09-story-001.md)

## Homologação

O teste de homologação usa o tópico SNS, a fila Standard, a DLQ e a role da tarefa ECS definidos para o ambiente. Após conferir a configuração e as permissões, use as saídas do stack e habilite `EventStructuralLock__Enabled` na API. Verifique uma confirmação de texto e uma confirmação de mídia em eventos distintos, a latência até a trava visível, a remoção do convite, a preservação da abertura e a fila de mensagens. [Configuração do transporte](../../../scrapbook-infrastructure/README.md#travamento-após-primeiro-registro)

A flag fica desligada por padrão. Sua desativação suspende o processamento e preserva o estado persistido e as pendências. A F09-STORY-002 cobre a recuperação operacional e os resultados duráveis para a F13. O cartão registra os merges de `hlg` e o resultado da publicação em homologação. [Card no Trello](https://trello.com/c/cWad9rrV)
