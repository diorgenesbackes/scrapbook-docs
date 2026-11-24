# Entrega da F06-STORY-001

A primeira identificação foi validada em Docker e promovida seletivamente. O formulário exige nome ou apelido e concordância com o aviso, sempre no contexto do código público da F05. A prévia em loopback usa um aviso sintético. [Card no Trello](https://trello.com/c/RiJ79nyD) · [Contrato e acesso local](../dev-front/f06-story-001.md)

| Repositório | Versão em `dev` usada na validação integrada | Candidato em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `93d0bf84b3819e62f801f342ca9919f1d94fbddc` | `8576245df8ca75dc1e939c2ba91e20e02803cf9c` |
| `scrapbook-tests` | `455c07d236a953277c26b1e63875325266a1cbb2` | `60d9757246987a596f144edd9e592be4b1f94016` |
| `scrapbook-docs` | `ea2381cb4780fa2f02bb750f7497786987500def` para as evidências | Contrato, revisão e evidências da história promovidos de forma seletiva. |

A comparação das árvores completas confirmou que frontend e testes em `hlg` têm o conteúdo validado em `dev`. O serviço usado no Docker estava em `76fd9e084afcfb24ccb2521aaed6d61d56190a03`. Os unitários passaram com 55 casos no frontend e 125 no backend. Os 15 cenários de navegador passaram, assim como as verificações defensivas. No ensaio 4G emulado, a S17 ficou utilizável em até 1.768 ms. [Revisão](../code-review/f06-story-001.md) · [E2E](../test-e2e/f06-story-001.md) · [Aceite](../test-funcional/2026-11-24_11-10-34/2-execucao.md) · [Desempenho](../test-performance/f06-story-001.md) · [Segurança](../test-security/f06-story-001.md)

## Publicação e aceite em homologação

O workflow Publicar UI falhou em **Reservar a versão**, com código de saída 1. Os jobs de build e publicação foram pulados. O candidato da história está no Git e a publicação correspondente não ocorreu. [Workflow](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45411675017) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45411675017/job/138506843089)

Após publicar o candidato, conferir o SHA de `release.json`, a página pública da F05, os cabeçalhos e a indisponibilidade da rota de diagnóstico na origem pública. O ensaio funcional da S17 com aviso sintético usa o candidato em Docker. O aviso oficial, as entradas da F07 e a validação da F08 precedem o aceite do fluxo público completo. [Decisão Q03](../../design-system/decisions.md)
