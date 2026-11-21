# F01-STORY-004 · Resultado da carga local

O k6 v2.3.0 executou as duas fases em uma stack Docker descartável, com os commits de `dev` registrados para a [validação funcional](../../test-funcional/2026-11-21_10-12-11/1-plano.md). A origem foi o proxy da UI em loopback. Cada fase teve uma rodada com o mesmo registro sintético. [Perfil e limites](1-perfil.md)

| Fase | Rota | Requisições | p50 | p95 | p99 | Máximo |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| Saudável | `/` | 300 | 1,15 ms | 3,16 ms | 3,81 ms | 4,82 ms |
| Saudável | `/health/ready` | 300 | 3,84 ms | 7,57 ms | 8,56 ms | 24,59 ms |
| Recuperação | `/` | 801 | 1,85 ms | 3,05 ms | 3,88 ms | 11,28 ms |
| Recuperação | `/health/ready` | 801 | 5,90 ms | 8,60 ms | 13,16 ms | 2 s |

O gerador concluiu 300 e 801 iterações, com taxa agregada de 40,00 e 40,04 requisições por segundo. Os dois processos retornaram código 0, com 2.202 verificações aprovadas, zero falhas inesperadas e zero descartes. Durante a recuperação, a página respondeu `200` em todas as 801 amostras. A prontidão respondeu `503` em 68 amostras e `200` nas outras 733. As respostas esperadas de indisponibilidade foram incluídas nos percentis da prontidão.

A primeira prontidão `200` foi observada 0,587 s após o comando de retomada do PostgreSQL, com a carga em andamento. A vitalidade permaneceu `200` na sonda de interrupção e o registro sintético foi preservado. O resultado aprova o perfil local exercitado. A medição em homologação fica com a pessoa responsável pelo projeto. [Ordem da recuperação](../../delivery/f01-story-004-recuperacao.md)
