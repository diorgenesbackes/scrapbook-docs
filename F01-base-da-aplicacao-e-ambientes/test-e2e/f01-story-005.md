# F01-STORY-005 · Regressão ponta a ponta local

`python3 -B docker_stack.py --browser` executou a suíte em uma cópia dos SHAs integrados de `dev` com Docker Compose descartável, portas em loopback e massa sintética. O cenário integrado terminou em 47,275 s com código 0. A prontidão foi `503` antes da migração, `200` após o preparo, `503` durante a interrupção do PostgreSQL e `200` após a recuperação. A suíte confirmou preservação de dados e volumes, comportamento do proxy, correlação e diagnóstico sanitizado.

| Projeto | Navegador | Casos aprovados |
| --- | --- | --- |
| Desktop Chromium | 158.0.8127.18 | 2 |
| Desktop Firefox | 159.0 | 2 |
| Desktop WebKit | 27.2 | 2 |
| Android emulado | Chromium 158.0.8127.18 | 2 |
| iPhone emulado | WebKit 27.2 | 2 |

O Playwright verificou estrutura semântica, reflow, foco por teclado e recuperação depois da indisponibilidade. Foram 10 casos aprovados, zero falhas e zero skips. A execução da UI com HTTPS e Amplify depende do ambiente implantado. [Procedimento](../delivery/f01-story-005-producao.md)
