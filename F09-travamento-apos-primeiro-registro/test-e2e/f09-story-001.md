# F09-STORY-001 — Testes de ponta a ponta

Em 26/11/2026, a suíte executou em Docker local com PostgreSQL e SNS/SQS emulados, usando a API em `4f3f024`, UI em `57f4e63`, testes em `7138d3d`, infraestrutura em `56759f3` e Liquibase em `a9be70d`. O comando `python3 -B event_structural_lock_stack.py --browser` terminou com código `0` em 66,275 s. A etapa Chromium aprovou os dois cenários em 3,6 s, sem retries nem casos pulados. A limpeza confirmou a remoção dos nove recursos descartáveis observados. [Runner](../../../scrapbook-tests/event_structural_lock_stack.py)

| Cenário | Resultado |
| --- | --- |
| S11 | Uma sessão convidada confirmou o primeiro registro. A sessão responsável observou o conflito da edição já aberta, exibiu a mensagem de trava, retirou os controles estruturais e conseguiu salvar a alteração de abertura ainda autorizada. |
| S13 e S14 | A primeira confirmação retirou QR, link e ações de convite da sessão responsável. A sessão destinatária perdeu a ação de aceitar após a reconsulta. O aceite por uma aba desatualizada retornou `409` e exibiu mensagem neutra. |
| Integração HTTP | Texto e foto confirmados geraram trava. A foto rejeitada foi excluída do fluxo. Reentrega e concorrência preservaram um único efeito, incluindo edição e aceite que terminaram antes da trava. |

Uma ambiguidade no seletor de boas-vindas interrompeu o preflight de S11 antes da primeira confirmação. O seletor corrigido limitou a busca ao texto exibido. O reteste integrado passou. Uma correção subsequente de correlação na API foi verificada em unitários e nos cenários HTTP pertinentes em Test. [Caso Playwright](../../../scrapbook-tests/event-structural-lock/event-structural-lock.spec.ts)
