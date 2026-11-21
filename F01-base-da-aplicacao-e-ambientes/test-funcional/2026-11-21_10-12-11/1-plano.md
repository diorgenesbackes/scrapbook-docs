# F01-STORY-004 · Plano de validação local antes de homologação

Validar os commits integrados em `dev` da API (`32cb2cd`), UI (`8919ad1`), Liquibase (`a460f08`), testes (`bfb23b7`) e infraestrutura (`cd0d5c4`). Usar stack descartável, PostgreSQL 18.6 e Liquibase 4.33.0 com contas e credenciais sintéticas. [Story no Trello](https://trello.com/c/eVGjsc6t) · [Decisões da F01](../../spec.md#3-decisões-técnicas)

| Critério | Cenário local e resultado esperado |
| --- | --- |
| CA1 | Na simulação, retomar o digest e o ZIP de uma publicação conhecida. Rejeitar divergência de origem, digest ou ZIP. Confirmar o estado ativo por digest, prontidão e manifesto. |
| CA2 | No PostgreSQL descartável, a prévia mantém o catálogo e o histórico. A reversão recusa alvo divergente e banco com dados. A base vazia regride, mantém o schema e os controles e permite reinstalação. |
| CA3 | Na simulação, a tentativa malsucedida fica registrada e os controles impedem ativação incompatível. No laboratório, observar as transições de prontidão e confirmar a recuperação do banco sem perder registros sintéticos. |

O ambiente real, incluindo acionamento dos workflows e retomada no ECS e Amplify, é escopo da verificação conduzida pela pessoa responsável a partir de `hlg`. [Procedimento de recuperação](../../delivery/f01-story-004-recuperacao.md)
