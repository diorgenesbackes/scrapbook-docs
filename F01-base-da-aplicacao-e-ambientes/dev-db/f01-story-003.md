# F01-STORY-003 · Imagem Liquibase no Fargate

A imagem usa o Liquibase 4.33.0 e JDK fixados por digest. Ela compila a pré-validação JDBC já usada na suíte local, valida e aplica o changelog e confere se a geração ativa do catálogo tem a assinatura esperada. A mudança preserva o changelog e a identidade do migrador. A tarefa usa o bundle RDS regional fixado e TLS com verificação de hostname.

O workflow publica o marcador da geração somente após o contêiner terminar com sucesso e com o digest candidato. A recuperação manual fica na F01-STORY-004. [Ordem de publicação](../delivery/f01-story-003-publicacao.md)
