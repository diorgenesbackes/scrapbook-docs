# F03-STORY-002 · Entrega da política de abertura

A história oferece a escolha entre abertura imediata e futura na criação e no formulário próprio do evento. Aplica a trava de Brasília, controla alterações concorrentes e grava a auditoria na mesma transação. O catálogo ativo precisa conter a geração `f03-story-001-event-creation`. [Card](https://trello.com/c/4CNJpCP3) · [Dependência e geração](f03-story-001.md)

## Validação local

Passaram 70 testes unitários .NET, 13 testes Vitest e 25 casos Playwright, incluindo as regressões de criação e da fundação. Os testes de integração comprovaram o conflito entre duas sessões e a reversão da atualização quando a auditoria falha. O checklist foi atualizado conforme a validação em cada etapa. [Revisão](../code-review/f03-story-002.md) · [Aceite](../test-funcional/f03-story-002.md) · [E2E](../test-e2e/f03-story-002.md) · [Segurança](../test-security/f03-story-002.md) · [Performance](../test-performance/f03-story-002.md)

## Homologação

| Repositório | `dev` validada | `hlg` confirmada | Resultado |
| --- | --- | --- | --- |
| API | `9c37192` | `245c1e3` | Falha na reserva de versão. [Execução](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/44915904702) |
| UI | `479d77e` | `5b7a633` | Falha na reserva de versão. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/44915903059) |
| Testes | `fb79401` | `376b7d5` | Validação local concluída. |

A reserva falhou antes dos jobs de publicação na AWS. O aceite em homologação segue pendente. [Configuração da publicação](../../F01-base-da-aplicacao-e-ambientes/delivery/f01-story-003-publicacao.md#configuração-do-github-e-aws)

Confirme a geração do Liquibase, a prontidão da API e o artefato da UI antes dos ensaios. O aceite remoto da jornada autenticada depende também da sessão real da F02. [Roteiro de homologação](f03-story-001.md#homologação)
