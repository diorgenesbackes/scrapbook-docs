# Plano — CRUD HTTP de Gestão de Eventos

Leva: GE-035…GE-046 e GE-048…GE-059. Selecionar todos os cartões em Todo antes da implementação. A GE-047 já foi implementada/validada junto da projeção na leva anterior.

Validar criação transacional com slot 1 e auditoria; isolamento/404 indistinguível; paginação/cursor protegido com conta, ordem, TTL e restart; detalhe; edições estruturais/de abertura; comparação de versão e no-op; retorno final dos triggers; cancelamento de obrigações antigas; budgets; body/JSON estritos; correlação e OpenAPI.

Executar manualmente HTTP/PostgreSQL em scrapbook-tests, com fixtures válidas e alvos locais, além dos unitários. Forçar falhas e conflitos, preservando a massa fora do cenário e o histórico Liquibase. A carga H-03 completa será executada com as telas integradas e 200 eventos.

Falha: retornar a tarefa afetada para Doing, corrigir e retestar antes de To Deploy. Não promover para main/hlg nem executar deploy.
