# Candidato da F08-STORY-002

O convidado pode enviar uma foto revisada com legenda, acompanhar bytes enviados, retomar com a mesma tentativa e receber confirmação após a preservação do original. A integração local verificou a evidência da versão exata, a borda das 13h, a recuperação do commit e o saneamento dos temporários. [Cartão](https://trello.com/c/EeNKYoWn) · [Aceite local](../test-funcional/2026-11-25_14-54-00/2-execucao.md)

| Repositório | SHA do candidato local | Base `dev` |
| --- | --- | --- |
| `scrapbook-service` | `20097b044e77ab9ef9cba408dd055c0c4b2884f7` | `b0ede23` |
| `scrapbook-ui` | `c8005e7847e2eaa326552d984137c808ebb859ff` | `9c4a570` |
| `scrapbook-tests` | `068add66e1b88818be3f85324bf7fb0e1755bb96` | `1623a1b` |
| `scrapbook-liquibase` | `ef51987c7c4a32722009e35f2cae6e5480762364` | `f5d1632` |
| `scrapbook-infrastructure` | `8d9a753c437193b32bca2dda0737216ea75fd865` | `f555467` |
| `scrapbook-docs` | Documentação da mesma branch da story | `c5d084c` |

O candidato passou em 305 unitários, quatro cenários Playwright com foto, integração HTTP/PostgreSQL/Moto e regressão do envio de texto. A suíte do banco exercitou 26 casos. Nove testes da infraestrutura e o `cfn-lint` passaram após a revisão de KMS. [E2E](../test-e2e/f08-story-002.md) · [Segurança](../test-security/f08-story-002.md) · [Revisão](../code-review/f08-story-002.md)

O perfil de 200 envios por foto registrou falhas de conexão e nenhum PUT completo. O runtime local do Docker ficou indisponível durante a verificação e a limpeza. O ensaio de carga requer reteste após a recuperação da stack descartável. [Carga](../test-performance/f08-story-002.md)

## Integração e homologação

O fluxo usa `feature/F08-STORY-002` nos seis repositórios e requer PR para `dev`. O cartão está em Doing enquanto a sessão do MCP não dispõe de `SCRAPBOOK_GITHUB_MCP_TOKEN`. Após os merges em `dev`, validar a versão integrada e promover o conteúdo selecionado por PR para `hlg`. O checklist de Test reúne as validações do resultado integrado.

Em homologação, exercitar a notificação nativa com `VersionId` e horário do S3, CORS/CSP, IAM/KMS, proteção da cópia privada e exclusão por versão. Validar a restrição de rede e arquivos do decoder e o orçamento de recursos da tarefa. A ativação de fotos começa desabilitada. [Infraestrutura](../dev-infra/f08-story-002.md)
