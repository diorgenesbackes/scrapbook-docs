# F01-STORY-003 · Teste de navegador da fundação local

O runner `python3 -B docker_stack.py --browser` terminou com código 0 em uma cópia temporária dos checkouts locais. O laboratório criou seu próprio projeto Docker Compose, usou portas de loopback e aplicou o Liquibase ao banco descartável. O Playwright usou os motores já instalados para a versão fixada pelo lockfile. A configuração temporária acrescentou apenas a emissão de um relatório JSON. [Runner local](../../../../scrapbook-tests/README.md) · [Spec da F01](../../spec.md#7-estratégia-de-testes)

| Projeto | Motor observado | Resultado |
| --- | --- | --- |
| Chromium desktop | 158.0.8127.18 | 2 passaram |
| Firefox desktop | 159.0 | 2 passaram |
| WebKit desktop | 27.2 | 2 passaram |
| Android emulado | Chromium 158.0.8127.18 | 2 passaram |
| iPhone emulado | WebKit 27.2 | 2 passaram |

O relatório JSON registrou dez aprovados, zero falhas, zero casos ignorados e zero casos intermitentes. Os cinco projetos verificaram estrutura semântica, reflow, navegação por teclado, foco e retorno de rota ausente. A análise automatizada do Axe não retornou violações nos cenários executados. No teste de recuperação, o Playwright interceptou a primeira resposta de prontidão para produzir a indisponibilidade. O usuário repetiu a ação por teclado e recebeu o estado disponível da API local.

A execução cobre o shell da fundação no ambiente local. A navegação no Amplify, dispositivos físicos e avaliação manual com leitor de tela seguem fora das evidências desta rodada. [Plano funcional](../../test-funcional/2026-11-20_18-52-46/1-plano.md)
