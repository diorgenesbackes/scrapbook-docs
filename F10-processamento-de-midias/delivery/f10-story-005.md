# Candidato de ativação da F10

Entrega de 28/11/2026 vinculada à [F10-STORY-005](https://trello.com/c/VIdGX9ti) e às gerações integradas de F10-STORY-001 a F10-STORY-004. A lista, os PRs de promoção e os respectivos SHAs de merge ficam registrados no cartão.

## Integração em dev

| Repositório | PR | SHA de merge |
| --- | --- | --- |
| Serviço | [#25](https://github.com/diorgenesbackes/scrapbook-service/pull/25) | `d7ac67dc9095ea55c1a4c402e91e9b5a2568e43c` |
| Infraestrutura | [#22](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/22) | `0ec49e6b121078d427bfe2c26829fe36346ff0a7` |
| Testes e correções das fixtures | [#25](https://github.com/diorgenesbackes/scrapbook-tests/pull/25) · [#26](https://github.com/diorgenesbackes/scrapbook-tests/pull/26) · [#27](https://github.com/diorgenesbackes/scrapbook-tests/pull/27) | `c1782eb0636d7038680c59654ea361c60299fe82` · `ab30cec12da3f2fb66b81bfe56d87fd509dad438` · `1157ee1a4f424baa7e9fec816e2571e262796365` |
| Procedimento na documentação | [#35](https://github.com/diorgenesbackes/scrapbook-docs/pull/35) | `bd724ecb50b9f0e62e81a629a81a252a3e032274` |

## Validações e publicação

- [Revisão técnica](../code-review/f10-story-005.md) · [Testes E2E](../test-e2e/f10-story-005-dev.md) · [Desempenho local](../test-performance/f10-story-005-dev.md) · [Verificação defensiva](../test-security/f10-story-005-dev.md)
- No laboratório, os 200 envios receberam recibo e 163 de 170 arquivos tiveram commit de disponibilidade comprovado dentro do limite monotônico de 72,751 s. Todos os 170 estavam disponíveis em até 279,366 s pelo mesmo método. O cálculo individual de latência ficou inconclusivo por descontinuidade no relógio da VM. A promoção usa a evidência conservadora da meta local e mantém abertos o aceite AWS e a medição exata de prontidão.
- O ensaio depende de uma versão da API confirmada no ECS e da geração `f10-story-004-media-recovery` aplicada pelo Liquibase. No diagnóstico inicial, os workflows da API `e1d32de` e do Liquibase `85ac012` falharam em **Reservar a versão** e pularam a publicação. A sessão também ficou sem identidade AWS válida no STS. [Workflow da API](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/46170864039) · [Workflow do Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/actions/runs/46170802261)
- Para o aceite em homologação, o responsável pelo ambiente confere os artefatos publicados e executa a sequência de habilitação e pausa. O ensaio real verifica acesso a versões e escopo IAM/KMS, entrega de eventos e alarmes, codecs e variantes de cada perfil, pressão sobre o recibo F08, carga mista e bordas, e preservação de dados após retomada e rollback. [Procedimento de homologação](../dev-infra/f10-story-005.md)
