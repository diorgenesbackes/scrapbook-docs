# F09-STORY-002 — Jornada de recuperação em Chromium

O cenário CA3 usa as duas sessões no navegador e o runner descartável, com falha controlada da gravação da auditoria da trava. A sessão convidada confirma o primeiro registro e recebe o recibo. A sessão responsável observa a edição enquanto o efeito está pendente. Após a retomada, a sessão consulta a versão atual, exibe o aviso neutro, retira os campos e as ações de convite e conserva a abertura permitida. [Cenário](../../../scrapbook-tests/event-structural-lock/event-structural-lock.spec.ts) · [Runner](../../../scrapbook-tests/event_structural_lock_recovery_stack.py)

A primeira rodada integrada usou API `8bda7e9`, infraestrutura `b86dc79`, suítes `26ee890`, UI `57f4e63` e Liquibase `a9be70d`. O recibo e a configuração anterior à trava foram observados. O runner encerrou com falha após aguardar 22 s pelo resultado terminal e o Chromium expirou aguardando a retomada. O inventário confirmou a limpeza da stack.

O ciclo técnico preservou a exigência de resultado terminal automático na suíte HTTP. A jornada do navegador passou a reconciliar o envelope legítimo da DLQ antes de exigir o único resultado terminal e retomar o primeiro fato. O orçamento maior se aplica ao ensaio com falha injetada. [PR da correção](https://github.com/diorgenesbackes/scrapbook-tests/pull/13) · [Aceite HTTP](../test-funcional/f09-story-002.md)

O reteste das suítes `f0fc3b7` executou `python3 -B event_structural_lock_recovery_stack.py --browser` nas demais versões de `dev` já indicadas. O runner terminou com código `0` em 52,073 s. O Chromium aprovou o único cenário de recuperação em 23,8 s, sem retries nem skips. A jornada confirmou a exibição do recibo durante a falha, a retomada, a remoção dos campos e das ações de convite e a abertura ainda habilitada. O runner verificou um único efeito durável.

A regressão `python3 -B event_structural_lock_stack.py --browser` passou em uma nova stack. O runner terminou em 40,565 s e o Chromium aprovou S11, S13 e S14 em dois cenários, com 3,15 s no navegador, sem retries, skips ou instabilidade. O inventário retornou ao inicial após as duas rodadas.
