# Plano: integração de todas as branches pendentes em main

Solicitação do responsável: integrar todas as branches ainda não incorporadas. [Auditoria](auditoria-antes.json) inclui refs locais e origin nos seis repositórios, árvores limpas e workflows dos candidatos.

## Escopo e ordem

- Atualizar origin e executar pull --ff-only nas seis mains.
- Integrar PP-STORY-005 antes de PP-STORY-006 em service, UI, tests, infrastructure e docs, com --no-ff.
- Liquibase já atualizado, sem branch pendente ou alteração de schema.
- Conferir árvores integradas contra as features validadas, build/unitários e documentação.
- Enviar mains, atualizar registros das histórias e auditar novamente todos os refs.

## Limites

Preservar branches, evidências e volumes. Não iniciar serviços. Merge não comprova deploy; nenhum workflow de publicação existe nos candidatos auditados. PP-002-029 e PP-003-034 continuam como QA humano pendente nas histórias anteriores.
