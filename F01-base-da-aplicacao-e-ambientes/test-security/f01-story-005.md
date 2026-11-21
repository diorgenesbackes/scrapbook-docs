# F01-STORY-005 · Segurança da base integrada

| Perspectiva | Controles exercitados em `dev` | Resultado |
| --- | --- | --- |
| White box | Separação dos jobs de PR e publicação, escopo OIDC, referências fixadas, recuperação vinculada à origem e SSM por ambiente. Casos simulados de marcador, segredo, branch e artefato divergentes. | As simulações recusaram os alvos e artefatos incorretos. |
| Grey box | Identidade DML no PostgreSQL descartável e configuração das tarefas locais. | As 13 regras de permissões conferidas corresponderam à matriz. API e UI executaram sem root, com filesystem de aplicação somente leitura, capacidades removidas e portas em loopback. O startup incompatível foi recusado com diagnóstico sanitizado. |
| Black box guiada | HTTP no loopback, caminhos de fallback e de arquivos ausentes, cabeçalhos, cookies, sondas e correlação. | Os status e cabeçalhos esperados foram observados. As requisições sintéticas com parâmetros privados não expuseram seus valores nos logs. |

O isolamento real entre ambientes, os principals IAM, o comportamento de WAF e os objetos S3/KMS requerem recursos e identidades autorizadas dos dois ambientes. A comparação e as recusas reais estão especificadas no roteiro de preparação. [Matriz de isolamento](../dev-infra/f01-story-005-producao.md#comprovar-isolamento)
