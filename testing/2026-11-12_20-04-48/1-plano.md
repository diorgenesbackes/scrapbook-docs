# Plano — preparação de Gestão de Eventos

Escopo: GE-001…GE-008 da [story](https://trello.com/c/kIAhFOno), conforme [planejamento](../../fdd/FDD-Gestao-de-Eventos/1-planejamento.md). Os cenários estavam nos cartões e runners antes da execução; este arquivo consolida seu registro após a execução.

Verificar restore/build/unitários, grants DML e DDL negado, repetição/colisão/reversão das fixtures, rollback da transação evento/membro/auditoria, keys privadas após recriação do provider, contrato HTTP de saúde e sua consulta no navegador desktop/móvel emulado. Conferir exclusão de libs/segredos no Git.

Executar baseline de /health com 10 requisições concorrentes por 5s após aquecimento. Essa operação não acessa PostgreSQL e não representa o perfil H-03 de Events.

Critério da leva: nenhum efeito parcial nos testes, dados/volume preservados, checks aprovados e evidência vinculada. O CRUD, a identidade na API, o cursor HTTP, a concorrência da feature e o perfil de 200 eventos por 60s dependem das próximas levas.
