# F09-STORY-002 — Entrega da recuperação de travas

O fluxo permite reconciliar a origem de uma entrega esgotada e retomar apenas o primeiro fato selecionado, mantendo o `messageId`. A retomada preserva a confirmação, os efeitos já aplicados e os resultados duráveis de falha e sucesso para a F13. [Cartão](https://trello.com/c/fzQlYzmM) · [Procedimento operacional](../dev-infra/f09-story-002.md)

## Integração em `dev`

| Componente | PR | SHA do merge |
| --- | --- | --- |
| API | [#15](https://github.com/diorgenesbackes/scrapbook-service/pull/15) | `8bda7e9cc7016d73fffbecb51c36fd80268dea6b` |
| Infraestrutura | [#11](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/11) | `b86dc7919cc4719ac3f19227897e34e28959928f` |
| Suítes | [#12](https://github.com/diorgenesbackes/scrapbook-tests/pull/12) | `26ee8909a4d810cf20669b6d896ff719e8e75490` |
| Reteste Chromium | [#13](https://github.com/diorgenesbackes/scrapbook-tests/pull/13) | `f0fc3b7493c33a9819c5adc769f3a71173c77bc6` |
| Precisão da fixture mista | [#14](https://github.com/diorgenesbackes/scrapbook-tests/pull/14) | `fa79c4f44bba7676bf885ff2242c4dd245f861bd` |
| Documentação da implementação | [#20](https://github.com/diorgenesbackes/scrapbook-docs/pull/20) | `6863321abbbcc123e765fed539f0c1767c21ea40` |

## Evidências do candidato

- [Revisão técnica](../code-review/f09-story-002.md)
- [Validação E2E](../test-e2e/f09-story-002.md)
- [Aceite funcional e integração](../test-funcional/f09-story-002.md)
- [Carga e drenagem](../test-performance/f09-story-002.md)
- [Segurança defensiva](../test-security/f09-story-002.md)

## Homologação

A promoção Git depende da F09-STORY-001. O aceite do ambiente usa o candidato da API com a flag explicitamente habilitada e o stack de mensageria configurado. Confirme separadamente a publicação, a aplicação dos grants, o isolamento das roles, o fluxo de inspeção e retomada no SNS/SQS real e o recebimento dos alarmes no canal operacional. Use uma primeira confirmação sintética para observar a trava e confronte os resultados por SHA. [Configuração e retomada](../dev-infra/f09-story-002.md)

A flag desligada suspende o processamento e preserva os estados duráveis. Os resultados para a F13 continuam na Outbox até a habilitação do seu encaminhamento. Os merges de `hlg` e o estado da publicação ficam registrados no cartão. [Trello](https://trello.com/c/fzQlYzmM)
