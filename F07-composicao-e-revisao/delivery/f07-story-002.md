# Entrega da F07-STORY-002

Depoimento e momento permitem selecionar e revisar fotos na janela da F05. A revisão conserva ordem, legenda do conjunto e identificação do mesmo evento. O limite de 20 MB foi definido como 20.000.000 bytes por foto. [Card](https://trello.com/c/djeVwk02) · [Contrato](../dev-front/f07-story-002.md)

| Repositório | Candidato testado em `dev` | Candidato em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `9c7aa7cb67a08e06aa430d647e237be5b57c8931` | `48fd0ad1f696227aea2384367ad435613a3cd76e` |
| `scrapbook-tests` | `8c8dfcfdee81251d99677580368961f1108c3bb1` | `ada93818f2d629ffbbb79fa03a328eb8d6e6f49d` |
| `scrapbook-docs` | Contrato e evidências da história | Os mesmos documentos promovidos seletivamente. |

As árvores completas da UI e dos testes coincidiram entre `dev` e `hlg`. Passaram 87 unitários da UI, 125 do serviço e 55 cenários de navegador. O seletor ficou utilizável em até 1.889 ms em 4G emulado. A prévia das dez fotos ficou disponível em até 486 ms no perfil medido. A amostra HEIC foi exibida em WebKit e iPhone emulado. Os demais perfis forneceram recuperação com a seleção válida preservada. [Revisão](../code-review/f07-story-002.md) · [Navegador](../test-e2e/f07-story-002.md) · [Aceite](../test-funcional/2026-11-24_14-12-15/2-execucao.md) · [Desempenho](../test-performance/f07-story-002.md) · [Segurança](../test-security/f07-story-002.md)

## Publicação e aceite em homologação

GitHub Actions falhou em Reservar a versão. Os jobs de build e publicação foram pulados. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45446477723) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45446477723/job/138631525846)

Após publicar o candidato, conferir o SHA em `release.json`, `blob:` em `img-src`, o ingresso de momento, as quantidades, a legenda e a recuperação de prévia em dispositivos físicos. A F08 integrará a validação autoritativa e a confirmação do conjunto. [Runner](../test-e2e/f07-story-002.md)
