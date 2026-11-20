# Plano de segurança local da F01-STORY-002

A avaliação cobre a F01-STORY-002 integrada na `dev` local. Atores disponíveis são visitante anônimo, identidade sintética de desenvolvimento em teste unitário e role DML em PostgreSQL descartável. As superfícies exercitadas são UI, proxy, saúde, startup e privilégios de banco. Contas públicas, sessões, eventos, upload, Cognito, IAM, S3 e WAF pertencem a outras entregas ou alvos. Os controles do projeto orientam os resultados. ASVS e WSTG orientam o método. [Spec](../../spec.md) · [HLD](../../../HLD.md#segurança) · [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) · [OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/)

| ID | Método | Evidência esperada |
| --- | --- | --- |
| SEG-01 | White box | Revisar configuração de autenticação sintética, seleção da escuta, layout de segredos, Docker, proxy e logs. Verificar dependências e mudanças Git com as ferramentas disponíveis. |
| SEG-02 | Grey box | Conferir default desabilitado, associação da identidade sintética válida nos unitários, recusa de credencial inválida e rejeição de habilitação fora de Development na stack. |
| SEG-03 | Grey box | Conectar com a role de aplicação. Validar acesso DML previsto, ausência de CREATE no schema, de associação à proprietária e de leitura do histórico. Conferir auditoria com INSERT e sem SELECT, UPDATE, DELETE e TRUNCATE. |
| SEG-04 | Black box guiado | Consultar por HTTP anônimo o início, fallback, recursos ausentes e saúde. Conferir CSP, proteção de enquadramento, política de referência, permissões, cache, cookies e correlação. |
| SEG-05 | Grey box e black box guiado | Conferir portas publicadas e processos de API/UI. Observar startup inválido e ausência de valores sintéticos em logs. Uma query de teste comum deve ficar fora dos logs do proxy e da API. |

Os ensaios de HTTP usam apenas GET comuns, sem autenticação externa, e a verificação de configuração usa duas execuções de startup recusadas. A intervenção em PostgreSQL limita-se à instalação explícita do schema da F01 no projeto descartável e a consultas de metadados. O relatório distingue análise estática de evidência dinâmica.
