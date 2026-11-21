# F01-STORY-003 · Imagem Liquibase no Fargate

A imagem usa o Liquibase 4.33.0 e JDK fixados por digest. Ela compila a pré-validação JDBC já usada na suíte local, valida e aplica o changelog e confere se a geração ativa do catálogo tem a assinatura esperada. A mudança preserva o changelog e a identidade do migrador. A tarefa usa o bundle RDS regional fixado e TLS com verificação de hostname.

Execute o bootstrap como administrador do banco com `CREATEROLE` e autoridade de administração sobre as roles Scrap Book quando elas já existirem. A preparação assume a role proprietária para aplicar os privilégios no schema e retira apenas o acesso temporário concedido ao operador na mesma transação. A suíte local usa um administrador `NOSUPERUSER` e verifica a preservação das opções de uma associação preexistente. [Bootstrap](../../../scrapbook-liquibase/config/bootstrap.sql) · [Permissões de roles no PostgreSQL](https://www.postgresql.org/docs/18/role-attributes.html)

O workflow registra a execução pendente antes de iniciar a tarefa Fargate. A publicação da geração exige estado `STOPPED`, código de saída zero, ARN da revisão e digest esperados. A recuperação manual da publicação fica na F01-STORY-004. [Ordem de publicação](../delivery/f01-story-003-publicacao.md)
