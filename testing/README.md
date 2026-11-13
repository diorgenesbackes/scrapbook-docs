# Ciclos de testes

Execuções manuais usam `AAAA-MM-DD_HH-mm-ss/`, em `America/Sao_Paulo`, com `1-plano.md` e `2-execucao.md`. Resultados devem identificar escopo, revisão, comandos, ambiente, falhas e testes não executados. Não confundir emulação com aparelho real nem baseline operacional com carga de negócio.

| Ciclo | Escopo | Resultado |
| --- | --- | --- |
| [2026-11-12_20-04-48](2026-11-12_20-04-48/2-execucao.md) | Preparação GE-001…GE-008 | Checks e validação local aprovados; Events/carga de negócio pendentes. |
| [2026-11-12_20-36-51](2026-11-12_20-36-51/2-execucao.md) | Persistência/regras GE-009…GE-034 + GE-047 | PostgreSQL real, regras e fronteira local; CRUD HTTP e carga pendentes. |
| [2026-11-12_21-45-00](2026-11-12_21-45-00/2-execucao.md) | Casos de uso/HTTP, GE-035…GE-059 + GE-086 | Dez grupos HTTP/PG, atomicidade, concorrência, cursor e OpenAPI aprovados. |
| [2026-11-12_22-23-17](2026-11-12_22-23-17/2-execucao.md) | UI, instrumentação, regressão e H-03 | 77 backend/39 frontend, 14 jornadas; carga 200 eventos/10 concorrentes/60s aprovada, zero erros. |
| [2026-11-13_08-59-05](2026-11-13_08-59-05/2-execucao.md) | RF-STORY-001 — Controllers/conceitos/contexto único | 80 unitários, 14 jornadas, H-03 e privacidade aprovados; serviços parados. |
| [2026-11-13_10-03-49](2026-11-13_10-03-49/2-execucao.md) | Complemento RF — Domain puro, Services em Application | 82 unitários, 14 jornadas, HTTP/PG e H-03 aprovados; Domain sem frameworks, serviços parados. |
| [2026-11-13_11-09-48](2026-11-13_11-09-48/2-execucao.md) | Dockerfiles API/UI e stack Compose — cartão 6 | 86 backend/39 UI, 14 jornadas, persistência e H-03 (132195 requests/zero erros) aprovados. |
| [2026-11-13_12-02-01](2026-11-13_12-02-01/2-execucao.md) | Identidade visual — cartão 3 | 38 telas/178 estados, 14 testes e 20 pares de contraste aprovados; revisão visual pendente. |

Resultados anteriores são fotografias do escopo daquela leva; pendências de CRUD/carga ali descritas foram tratadas nos ciclos seguintes. Sem publicação/aceite humano declarado. [Matriz dos 26 AC](../fdd/FDD-Gestao-de-Eventos/3-testes.md).
