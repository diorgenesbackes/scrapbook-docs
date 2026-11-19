# Documentação do Scrap Book

Base documental preservada no reinício do projeto em 16/11/2026.

- [Produto e requisitos](PRD.md)
- [Arquitetura](HLD.md)
- [Guidelines](guidelines/README.md)
- [Design system](design-system/README.md)
- [Modelagem do banco](db/README.md)

As branches atuais são `main` e `dev`. Os documentos preservam o conteúdo anterior e podem conter referências ao histórico e a decisões que serão revistas durante a reconstrução.

## Documentação por feature

Identifique a pasta da feature pelo ID no PRD ou pela localização de `spec.md` e `plan.md`. Salve os documentos de desenvolvimento, revisão, entrega e testes em `scrapbook-docs/<feature>/<etapa>/`. Crie a subpasta da etapa quando houver um documento para registrar e use links relativos entre os documentos. Por exemplo, o relatório de code review da F01 fica em `F01-base-da-aplicacao-e-ambientes/code-review/`.

| Etapa | Subpasta |
| --- | --- |
| Desenvolvimento frontend | `dev-front/` |
| Desenvolvimento backend | `dev-back/` |
| Banco de dados | `dev-db/` |
| Infraestrutura | `dev-infra/` |
| Revisão de código | `code-review/` |
| Validação de design | `test-design/` |
| Testes ponta a ponta | `test-e2e/` |
| Validação funcional | `test-funcional/` |
| Testes de performance | `test-performance/` |
| Testes e revisão de segurança | `test-security/` |
| Entrega e promoção | `delivery/` |

Em trabalho com várias features, registre o que se aplica a cada uma em sua pasta e referencie evidências compartilhadas por links. Preserve relatórios e evidências de ciclos anteriores. Guarde evidências revisadas e sanitizadas com os documentos da etapa, mantenha resultados brutos sensíveis em armazenamento privado e mantenha código e suítes executáveis nos repositórios responsáveis.
