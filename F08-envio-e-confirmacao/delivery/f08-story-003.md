# Candidato da F08-STORY-003

O convidado pode enviar de uma a dez fotos de um momento com uma legenda comum, acompanhar o progresso e retomar somente as posições pendentes. O sistema publica um único recibo depois de confirmar o conjunto inteiro. [Cartão](https://trello.com/c/nvB32cdj) · [Resultados locais](../test-funcional/2026-11-25_20-40-42/2-execucao.md) · [Revisão](../code-review/f08-story-003.md)

| Repositório | SHA da branch da story | Dependência F08-STORY-002 |
| --- | --- | --- |
| `scrapbook-service` | `1d069665d99b2cebef6b07bb8d788de81b8f8d35` | `20097b0` |
| `scrapbook-ui` | `139b970d69b1ea44a626889567008506a51b0f7a` | `c8005e7` |
| `scrapbook-tests` | `0370314648def4d92c4692ba2c5e5e7e39456570` | `068add6` |
| `scrapbook-docs` | Revisão deste documento | `e415b9a` |

As branches `feature/F08-STORY-003` partem das versões publicadas da F08-STORY-002, que incluem os contratos de evidência, armazenamento versionado e migração. A ordem de integração é F08-STORY-002 seguida de F08-STORY-003. [Entrega da dependência](f08-story-002.md)

O candidato local aguardou a integração da dependência e a autenticação do MCP do GitHub. A validação da versão integrada e a promoção para `hlg` seguem pelo fluxo de PRs.

## Integração confirmada em dev

A F08-STORY-003 entrou em `dev` após a F08-STORY-002. Todos os checks de PR passaram. Liquibase e infraestrutura usam os SHAs integrados da dependência para o ensaio local.

| PR | SHA em dev |
| --- | --- |
| [`scrapbook-service`](https://github.com/diorgenesbackes/scrapbook-service/pull/2) | `c8c7ed9e2c46bceab965f646fd4a844112b3a166` |
| [`scrapbook-ui`](https://github.com/diorgenesbackes/scrapbook-ui/pull/2) | `a46fc573b3274fe73609e01b1be22d9729ecca91` |
| [`scrapbook-tests`](https://github.com/diorgenesbackes/scrapbook-tests/pull/2) | `81d85ea4a900e1bd820d95937503ddcfbb833751` |
| [`scrapbook-docs`](https://github.com/diorgenesbackes/scrapbook-docs/pull/5) | `1c0db3571729fbc9d197e1a2726715d400fef729` |

A integração HTTP e as seis jornadas de navegador passaram. Foram exercitadas a conclusão na fronteira do prazo, a retomada da posição pendente, a rejeição integral e a conferência de oito versões privadas no armazenamento emulado. As dez jornadas de regressão de depoimentos passaram. [Aceite integrado](../test-funcional/2026-11-25_22-38-00/2-execucao.md)
