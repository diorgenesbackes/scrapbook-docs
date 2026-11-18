# Validação da instalação inicial

Execução local da F01-STORY-001 em 18/11/2026. A suíte usou Liquibase 4.33.0, driver JDBC PostgreSQL 42.7.7 e PostgreSQL 18.6 em contêiner descartável, fixado por digest no manifesto de versões do repositório Liquibase. [História no Trello](https://trello.com/c/gSGvlzEZ)

## Resultado

`python3 tests/test_baseline.py` concluiu com código 0 e **8 cenários aprovados**. A suíte criou seus próprios alvos em loopback e removeu o contêiner ao final.

| Cenário | Evidência observada |
| --- | --- |
| Instalação em banco vazio | Validação do changelog e criação das 24 tabelas de aplicação, com 24 PKs identity e 34 FKs restritivas. Foi verificada a presença das colunas de lease e do índice parcial de recebimento. |
| Reaplicação e drift | A segunda aplicação preservou assinatura e histórico. Acrescentar uma coluna incompatível impediu a aplicação e o rollback sem remover dados ou tabelas. |
| Objeto preexistente | A instalação recusou o catálogo incompatível e preservou a tabela sintética existente. |
| Identidades e auditoria | A aplicação conseguiu gerar um ID e escrever dados nas tabelas autorizadas. DDL, alteração de role, acesso aos controles do Liquibase e leitura ou mutação da auditoria foram recusados. |
| Unicidade técnica | A Outbox recusou duplicação do identificador global. O controle de mensagens recusou duplicação no mesmo consumidor e aceitou outro consumidor para a mesma mensagem. |
| Falha em changeset posterior | O histórico da instalação permaneceu intacto. A criação anterior à falha na mesma transação sintética foi revertida. |
| Rollback vazio e reaplicação | A reversão removeu as tabelas de aplicação e a assinatura. Preservou o schema preparado e os controles, e a aplicação seguinte restaurou as 24 tabelas. |
| Rollback protegido | Dados sintéticos e dependência externa impediram a reversão. Inventário e histórico foram preservados nos dois casos. |

A primeira execução identificou que o cliente psql incluía o resumo do comando junto do ID durante a prova de permissões. Foi corrigida a leitura da saída e a suíte foi executada novamente por inteiro, com sucesso.

## Alcance

A validação cobre o changelog `f01-story-001-base` e as permissões locais de banco para o modelo inicial. Os procedimentos e os detalhes do catálogo estão no [modelo vigente](modelo-vigente.md). Configuração de ambientes AWS, validações da aplicação e promoção são acompanhadas nas histórias correspondentes.
