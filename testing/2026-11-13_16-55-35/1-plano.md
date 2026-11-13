# Plano: PP-STORY-004

Plano consolidado das 35 subtarefas da [disponibilidade dos envios](../../fdd/FDD-Disponibilidade-dos-Envios/FDD-Disponibilidade-dos-Envios.md). Implementação na feature própria, incluindo PP-STORY-003 como dependência; sem merge em main nesta rodada.

## Objetivos e critérios

Verificar not_open/open/closed pela hora do servidor, regra compartilhada de duas horas antes até 12h seguintes, tolerância sem reabertura e galeria independente. Cobrir instantes antes/no/depois, 12h30/13h/depois, madrugada e viradas de mês/ano/bissexto. Uma amostra UTC por resposta; marcos civis de Brasília e prazo de revalidação de até 30s.

UI não decide por Date.now/fuso do aparelho. Validar timers, cancelamento, foco/online/visibilidade, resposta antiga, tempo de transporte, alteração pela gestão e preservação da apresentação/QR durante falha. Anúncios sem mover foco, sem controles de captura sem destino.

## Ambiente e evidências

Unitários xUnit/Vitest nos projetos de aplicação. HTTP/Playwright em laboratório Compose com nomes/credenciais/portas/volumes exclusivos, mesmas 47 migrations e runtime DML. Host Kestrel adicional somente em probes, com controller/middleware/repositório/serviço reais e TimeProvider por arquivo privado. Não é o Program de produção completo; Compose verifica esse host separadamente. Sem endpoint para alterar hora ou relógio substituível na aplicação de produção.

Chromium Tokyo/WebKit Los Angeles emulados, Date fixada em 2099 e timers monotônicos reais. Só o GET público selecionado é encaminhado ao host temporal; a jornada de parada/retomada usa o Compose real, incluindo Nginx e API, restrito ao projeto descartável. Preservar volumes persistentes do usuário.

Regressões de compartilhamento, apresentação, acesso público e gestão. Perfil 4G: 10/2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cinco contextos novos/cache desligado, apresentação/capa/QR/disponibilidade prontos até 3s por amostra. Carga pública com 1004 eventos/dez clientes/60s, zero erros e p95 até 500ms.

Registrar revisão, comandos, resultados, falhas/retestes, capturas e transições observadas em UTC. Não inventar horas ativas, aparelho real, leitor de tela ou publicação. PP-003-034 e PP-002-029 continuam pendentes fora desta história. Sem DDL novo, captura/upload, autenticação ou AWS.
