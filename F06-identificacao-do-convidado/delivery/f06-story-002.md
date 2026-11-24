# Entrega da F06-STORY-002

A S17 reutiliza a identificação no mesmo evento, exige nova concordância ao editar o nome e mantém o rascunho da página quando o armazenamento falha. O candidato usa aviso e rascunho sintéticos nas prévias locais das duas categorias. [Card](https://trello.com/c/lW6dEG0h) · [Contrato](../dev-front/f06-story-002.md)

| Repositório | Candidato testado em `dev` | Promoção em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `68aeafd5c850e16acc2e7aa0ff357fc442dbf8de` | `3e00e9b5d2cc966704bd443a97d47f155deaee94` |
| `scrapbook-tests` | `ecb287705b808dc3e508250baccdbde42b8358a7` | `cc55f57e48329b9cfc9568a80dbde40c2a6c8d46` |
| `scrapbook-docs` | Contrato e evidências da story | Contrato e evidências promovidos seletivamente. |

As árvores completas do frontend e dos testes em `hlg` correspondem às validadas em `dev`. Passaram 75 unitários no frontend, 125 no backend, 35 casos Playwright e três cenários de desempenho. O tempo máximo observado foi de 1.770 ms no perfil 4G emulado. A verificação defensiva confirmou a separação por evento e a ausência de dados sintéticos nos logs. [Revisão](../code-review/f06-story-002.md) · [E2E](../test-e2e/f06-story-002.md) · [Aceite](../test-funcional/2026-11-24_11-53-47/2-execucao.md) · [Desempenho](../test-performance/f06-story-002.md) · [Segurança](../test-security/f06-story-002.md)

## Publicação e aceite em homologação

O GitHub Actions falhou em Reservar a versão. Os jobs de build e publicação foram pulados, e a versão correspondente ao candidato não foi publicada. [Workflow](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45419623901) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45419623901/job/138535248351)

Na publicação do candidato, conferir o SHA de `release.json`, a página pública da F05 e a indisponibilidade das prévias de diagnóstico no domínio público. O aceite da S17 com dados sintéticos pode usar o candidato em Docker com o [runner da história](../test-e2e/f06-story-002.md). A abertura do fluxo público depende do aviso oficial e das integrações F07 e F08.
