# F03-STORY-001 · Segurança local

| Perspectiva | Controles verificados |
| --- | --- |
| White box | Vínculo exigido antes da consulta de detalhe e replay, filtro de conta no SQL, proteção autenticada do cursor e correlação sem entrada de negócio nos logs. |
| Grey box | Duas contas sintéticas em instâncias da API separadas, mesmo PostgreSQL descartável. O cursor obtido na primeira conta foi recusado pela segunda e a partida com identidade local fora de Development foi recusada na regressão da fundação. |
| Black box guiada | `401` sem identidade, `404` igual para conta alheia e recurso ausente, `403` em POST de origem não autorizada, mídia incorreta e JSON duplicado recusados. A resposta protegida de erro incluiu `no-store`. |

O ensaio cobre o fluxo de eventos e a configuração local de desenvolvimento. A sessão real e o ambiente AWS dependem da integração correspondente. [Revisão técnica](../code-review/f03-story-001.md) · [Validação funcional](../test-funcional/f03-story-001.md)
