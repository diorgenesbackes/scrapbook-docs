# Desempenho local da prévia e do aceite

O runner usa até 12 requisições concorrentes. Cada linha contém 60 respostas válidas e usa a meta de referência de 500 ms no p95. A amostra de mesmo convite mede a primeira escrita e a recuperação concorrente do mesmo resultado. A de eventos distintos mede 60 aceites independentes.

| Perfil | Prévia p95 | Aceite p95 |
| --- | ---: | ---: |
| Mesmo convite | 31,21 ms | 31,54 ms |
| Eventos distintos | 21,22 ms | 14,90 ms |

Os resultados são do primeiro ensaio completo no Docker local. A corrida controlada com a trava é cenário de correção funcional e fica fora dessa amostra.
