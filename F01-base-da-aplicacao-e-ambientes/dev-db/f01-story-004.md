# F01-STORY-004 · Reversão da base inicial

O entrypoint mantém a aplicação do changelog no modo padrão e expõe dois comandos de recuperação para o banco descartável `scrapbook_recovery`: `preview-empty-base` e `rollback-empty-base`. O endpoint esperado deve coincidir com o host da URL JDBC verificada por TLS. O solicitante do workflow escolhe apenas a operação. O alvo, a definição ECS e as identidades vêm da configuração do ambiente de homologação. [Procedimento operacional](../delivery/f01-story-004-recuperacao.md)

O SQL de pré-validação verifica a geração e a assinatura vigentes, a identidade dedicada, um único changeset executado e o controle sem lock ativo. A reversão existente bloqueia as tabelas durante a verificação, recusa a presença de dados e segue com o DDL na transação. A conferência posterior exige catálogo e histórico vazios, com os controles Liquibase preservados.

A imagem inclui o bundle público verificado pelo hash original. O diretório recebe permissão de travessia para o usuário padrão do Liquibase. A imagem da API recebe o mesmo ajuste para corrigir o bloqueio identificado na validação da F01-STORY-003. [Evidência anterior](../test-funcional/2026-11-20_18-52-46/2-execucao.md)

O ensaio local confirmou a prévia, a proteção de dados existentes, a reversão da base vazia e a reinstalação. [Resultados](../test-funcional/f01-story-004-recuperacao-local.md)
