# Plano — jornadas da interface e instrumentação

Leva de implementação: GE-060…GE-085 (GE-086 já integrada à leva HTTP). Os 23 cartões de frontend foram selecionados em Todo antes de iniciar; a instrumentação segue o mesmo fluxo.

Validar navegação direta, vazio/carregando/erro, paginação e refresh, criação, dois salvamentos independentes, restauração de boas-vindas, conflitos, resposta perdida sem repetir mutação, indisponibilidade, teclado e layout móvel. Usar Chromium desktop e WebKit em emulação de iPhone, registrando os limites dessa emulação. Executar datas civis também em outro fuso.

Conferir JSON UTC, catálogo de logs, dimensões agregadas de métricas e spans sem dados privados. A evidência deve incluir unitários, HTTP/PostgreSQL e navegador, comandos/revisões e falhas/retestes. O laboratório usa container PostgreSQL exclusivo, aplica as 47 migrations existentes e remove somente seus processos/container ao terminar; o Compose e volume do usuário são preservados.

O perfil H-03 terá 200 eventos sintéticos, 10 concorrentes, aquecimento e 60s medidos; lista, detalhe e atualização usam versões válidas, com p95 por operação. Não extrapolar para uploads, AWS, dispositivos físicos ou disponibilidade global.
