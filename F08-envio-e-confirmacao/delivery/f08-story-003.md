# Candidato da F08-STORY-003

O convidado pode enviar de uma a dez fotos de um momento com uma legenda comum, acompanhar o progresso e retomar somente as posições pendentes. O sistema publica um único recibo depois de confirmar o conjunto inteiro. [Cartão](https://trello.com/c/nvB32cdj) · [Resultados locais](../test-funcional/2026-11-25_20-40-42/2-execucao.md) · [Revisão](../code-review/f08-story-003.md)

| Repositório | SHA da branch da story | Dependência F08-STORY-002 |
| --- | --- | --- |
| `scrapbook-service` | `1d069665d99b2cebef6b07bb8d788de81b8f8d35` | `20097b0` |
| `scrapbook-ui` | `3e3cde2f4b9d0a6a5b48de5bdc76c198fdf36d05` | `c8005e7` |
| `scrapbook-tests` | `0370314648def4d92c4692ba2c5e5e7e39456570` | `068add6` |
| `scrapbook-docs` | Revisão deste documento | `e415b9a` |

As branches `feature/F08-STORY-003` partem das versões publicadas da F08-STORY-002, que incluem os contratos de evidência, armazenamento versionado e migração. A ordem de integração é F08-STORY-002 seguida de F08-STORY-003. [Entrega da dependência](f08-story-002.md)

A sessão do MCP do GitHub aguarda credencial. A integração para `dev` e a promoção para `hlg` seguem por PR. O cartão permanece em Doing enquanto a dependência e a integração estão pendentes. Na versão integrada, executar a fixture completa, o E2E, o aceite funcional, a carga e as verificações defensivas antes da promoção.
