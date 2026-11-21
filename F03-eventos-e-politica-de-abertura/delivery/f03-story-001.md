# F03-STORY-001 · Entrega de criação e consulta

O candidato permite criar, listar e consultar eventos vinculados no ambiente local. A implementação parte da base da F01 e usa contas sintéticas de Development. A F02 fornece identidade e sessões reais na integração. [Card](https://trello.com/c/6LlSdQxc) · [Spec](../spec.md)

## Evidências locais

- O build e a revisão técnica passaram com 56 unitários .NET, 12 testes Vitest e 22 testes do Liquibase. [Revisão](../code-review/f03-story-001.md)
- A concorrência, o isolamento entre as duas contas, a paginação e a recuperação de resposta passaram no PostgreSQL e no navegador. Passaram cinco casos da story e dez casos da fundação depois do merge em `dev`. [Aceite local](../test-funcional/f03-story-001.md) · [E2E](../test-e2e/f03-story-001.md)
- A verificação de segurança local e o perfil de 160 requisições passaram. [Segurança](../test-security/f03-story-001.md) · [Carga](../test-performance/f03-story-001.md)

## Homologação

| Componente | Dev validada | Hlg promovida | Resultado |
| --- | --- | --- | --- |
| Liquibase | `f5d1632` | `a6c5901` | Falha na reserva. [Execução](https://github.com/diorgenesbackes/scrapbook-liquibase/actions/runs/44914342379) |
| API | `27b1da5` | `0a96ded` | Falha na reserva. [Execução](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/44914403476) |
| UI | `d156bbc` | `88b78ac` | Falha na reserva. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/44914403266) |
| Testes | `4564e00` | `6a93590` | Validação local. |
| Infraestrutura | `adf02f0` | `90461c0` | Validação local. |

Os jobs de publicação na AWS foram pulados. A consulta pública aos três repositórios retornou a lista de environments vazia. A reserva exige `RELEASE_SERIES` e permissões de tag configuradas. Os detalhes da falha dependem do acesso administrativo aos logs. [Configuração do GitHub e AWS](../../F01-base-da-aplicacao-e-ambientes/delivery/f01-story-003-publicacao.md#configuração-do-github-e-aws)

Antes de ativar a API, publique o Liquibase e confirme o marcador com a geração `f03-story-001-event-creation`. Configure `DATABASE_GENERATION` da API com a geração do candidato. A API anterior opera durante o avanço das colunas nulas. O rollback da geração recusa a remoção de chaves já gravadas. [Mudança no catálogo](../dev-db/f03-story-001.md) · [Publicação da fundação](../../F01-base-da-aplicacao-e-ambientes/delivery/f01-story-003-publicacao.md)

Para o aceite, confirme saúde, prontidão e build ativo. A criação autenticada com sessão real depende da F02. Registre a fronteira de isolamento, o replay e a paginação quando o candidato de autenticação estiver integrado.
