# F03-STORY-004 · Carga de consultas de abertura

k6 2.3.0 no host, serviço em Docker e dados descartáveis no PostgreSQL. Seis usuários virtuais consultaram um evento aberto e outro fechado, com duas contas vinculadas, dez segundos de duração e pausa de 200 ms por consulta. O relógio da fixture permaneceu em 02/01/2028 às 00h de Brasília. A meta foi p95 menor que 500 ms, com zero falhas no perfil.

| Métrica do candidato | Resultado |
| --- | --- |
| Requisições e taxa | 294 consultas, 28,82/s |
| p50 / p95 / máximo para estado aberto | 7,62 ms / 11,86 ms / 16,08 ms |
| p50 / p95 / máximo para estado fechado | 7,96 ms / 12,34 ms / 14,40 ms |
| Falhas | 0 de 294 (0%) |

Meta atendida no ensaio em loopback. A medição inclui a leitura do relógio de teste na cópia descartável do serviço.

[Entrega](../delivery/f03-story-004.md)
