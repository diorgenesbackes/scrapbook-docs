# F01-STORY-002 · Infraestrutura local

O Docker Compose publica UI, API e PostgreSQL somente em loopback. API e UI executam sem root, com sistema de arquivos de leitura e área temporária em memória. Os volumes nomeados preservam dados do PostgreSQL e chaves de proteção da API após recriação dos contêineres. [Compose](../../../scrapbook-infrastructure/compose.yaml)

O procedimento gera a configuração privada local, inicia PostgreSQL, prepara as identidades e aplica o changelog da F01-STORY-001 antes de iniciar API e UI. As execuções do Liquibase usam credencial separada da aplicação. O guia de operação descreve comandos de inicialização, diagnóstico e parada com volumes preservados. [Operação local](../../../scrapbook-infrastructure/README.md)

A validação criou um projeto Compose isolado com portas e credenciais sintéticas. O runner confirmou a publicação em loopback, a execução do Liquibase, a recuperação após indisponibilidade do banco e a preservação dos volumes. [Evidência de validação](../test-funcional/2026-11-19_11-07-50/2-execucao.md)
