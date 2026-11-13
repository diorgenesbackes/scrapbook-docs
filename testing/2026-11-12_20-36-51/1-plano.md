# Plano — persistência e regras de Events

Escopo: GE-009…GE-034 e GE-047 (incluída por compartilhar a projeção de capacidades com GE-018). Cenários previamente definidos nos cartões, unitários e probe; consolidação documental após a execução.

Verificar os quatro mapeamentos de Events e contexto separado de Identity, DI scoped, consulta por vínculo, validação de fixture e modos locais. Usar PostgreSQL 18.6 real com usuário DML, testando nome com 120 e mensagem com 2.000 escalares suplementares, criação/vínculo/auditoria em transação desfeita, valores finais dos triggers e no-op.

Nos unitários, cobrir normalização, formatos/overflow, 29/02, janelas 12h/13h, travamento estrutural, congelamento anterior à mudança de data, transição única de immediate e versões delegadas ao banco. Testar catálogo SQLSTATE sem depender de mensagens.

Verificar 403 para Origin não permitido, preflight explícito e recusa de startup com simulação fora de Development. Não declarar CRUD HTTP, cursor protegido completo ou carga/performance de Events aprovados nesta leva.
