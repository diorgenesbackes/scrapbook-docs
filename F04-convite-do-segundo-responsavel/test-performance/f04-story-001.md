# Desempenho local da emissão de convite

O ensaio usa PostgreSQL, interface, API e 12 requisições concorrentes na stack Docker. Mede separadamente 60 consultas e 60 emissões para o mesmo evento. Esse perfil força disputa pelo mesmo lock e usa a referência de 500 ms de p95 para operações comuns.

Na primeira execução completa, consulta teve p95 de 22,63 ms e emissão, p95 de 38,09 ms. Com uma transação de teste segurando o lock por 1,2 s, a consulta esperou e terminou em 1.207,10 ms. O bloqueio artificial mede o efeito da serialização fora da amostra de carga regular.

No resultado integrado em `dev`, os p95 foram 15,54 ms para consulta e 29,20 ms para emissão. A consulta sob o lock controlado terminou em 1.209,48 ms. As 120 operações da amostra regular foram concluídas.
