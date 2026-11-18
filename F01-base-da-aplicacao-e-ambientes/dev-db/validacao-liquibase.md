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

## Revalidação após o code review

`python3 tests/test_baseline.py` concluiu com código 0 e **14 cenários aprovados** em PostgreSQL 18.6 descartável e Liquibase 4.33.0. Os oito cenários originais continuaram passando e seis novos testes cobrem as regressões abaixo.

| Achado | Resultado observado |
| --- | --- |
| Privilégios preexistentes | O bootstrap recusou associação a uma role de leitura ampla e defaults de SELECT globais ou do schema para PUBLIC. A instalação abortou quando um grant padrão posterior ampliou as permissões de runtime. A reaplicação recusou SELECT indevido sobre uma coluna no controle do Liquibase. |
| Evolução de catálogo | Um changeset seguinte registrou sua geração na mesma transação do DDL. Repetir `update` preservou os dois registros no histórico. Reverter a evolução restaurou a assinatura e o histórico da geração anterior e permitiu novo `update`. |
| Sequências identity | Mudar o incremento da sequência bloqueou `update` e rollback, preservando as 24 tabelas e o histórico. Inserir dados e avançar a sequência normalmente preservou a validade da assinatura. |

## Revalidação após a segunda revisão

`python3 tests/test_baseline.py` concluiu com código 0. Os **15 testes** passaram com PostgreSQL 18.6 descartável, Liquibase 4.33.0 e driver JDBC PostgreSQL 42.7.7.

| Cenário | Resultado observado |
| --- | --- |
| Default indevido após o bootstrap | A pré-validação recusou SELECT e UPDATE padrão para `scrapbook_app` antes de iniciar a CLI. O banco ficou sem tabelas da aplicação, `databasechangelog` e `databasechangeloglock`. |
| Controle do Liquibase preexistente | A reaplicação recusou um grant de SELECT em uma coluna de `databasechangelog` e preservou o histórico. |
| Reversão incompleta | A assinatura divergente da geração de destino bloqueou o rollback. A transação preservou as duas colunas da evolução, a geração ativa e os dois registros do histórico. O `update` seguinte reconheceu a evolução aplicada. |
| Aplicação e reversão completas | Dois ciclos da mesma evolução preservaram as assinaturas históricas. Após cada rollback, o catálogo conferiu com a assinatura inicial e o histórico voltou a conter somente o changeset inicial. |

Uma verificação adicional confirmou os comandos documentados com descoberta automática do JDK e do driver. `update-sql` gerou o SQL sem criar tabelas de aplicação. Após a instalação, `status` informou que o banco estava atualizado e o histórico manteve um único registro.
