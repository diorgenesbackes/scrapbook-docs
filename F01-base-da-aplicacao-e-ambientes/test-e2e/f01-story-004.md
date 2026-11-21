# F01-STORY-004 · Regressão ponta a ponta local

`python3 -B docker_stack.py --browser` executou a suíte da fundação em uma cópia descartável das branches candidatas. A integração terminou com código 0 em 48,684 segundos. Conferiu proxy, cabeçalhos, vitalidade, prontidão, falha e retorno do PostgreSQL, preservação dos volumes e diagnóstico sanitizado.

Passaram os dez casos Playwright. A suíte exerceu estrutura semântica, reflow, foco por teclado e recuperação após indisponibilidade.

| Projeto | Navegador observado | Casos |
| --- | --- | --- |
| Desktop Chromium | 158.0.8127.18 | 2 |
| Desktop Firefox | 159.0 | 2 |
| Desktop WebKit | 27.2 | 2 |
| Android emulado | Chromium 158.0.8127.18 | 2 |
| iPhone emulado | WebKit 27.2 | 2 |

O laboratório comprova a regressão local. A ativação de um ZIP conhecido no Amplify e a confirmação da versão servida permanecem dependentes de um alvo de homologação identificado. [Procedimento de recuperação](../delivery/f01-story-004-recuperacao.md)

## Reteste local em 21/11/2026

A suíte foi repetida com os commits integrados em `dev` da API `32cb2cd`, UI `8919ad1`, Liquibase `a460f08`, testes `bfb23b7` e infraestrutura `cd0d5c4`. O ambiente Compose descartável terminou em 49,982 segundos, com código 0. Passaram os 10 casos Playwright nos mesmos cinco projetos, incluindo a recuperação após indisponibilidade.
