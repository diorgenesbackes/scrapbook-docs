# F09-STORY-001 — Revisão técnica

A revisão independente conferiu a transação de trava, a ordem dos locks, o vínculo com o primeiro fato persistido, a precisão temporal da F08, o ACK pós-commit e a configuração de transporte. A rodada de correções fechou sem achados no escopo revisado. [Spec](../spec.md) · [Plano](../plan.md)

| Achado inicial | Correção revisada |
| --- | --- |
| P2 · Consulta antiga reabria controles de abertura na mesma versão | A página preserva o estado temporal já observado na mesma versão. O unitário atrasa a consulta antiga até depois do 409 e da reconciliação. |
| P2 · URL HTTPS não representava necessariamente a fila regional | Em ambiente publicado, startup exige a URL canônica HTTPS de SQS com região e conta do tópico. O emulador mantém configuração de desenvolvimento. |
| P2 · Latência de carga antecedia a persistência da trava | A fixture descartável registra o timestamp de commit do PostgreSQL e confere a visibilidade por GET depois da transação. |
| P3 · Revalidação desmontava os controles de convite | As páginas preservam os controles durante a consulta, suspendem suas ações e removem o convite ao observar a trava. |
| P3 · Parâmetro de IAM com indicação imprecisa | O README identifica a role da tarefa ECS (`taskRoleArn`). |

| Componente | SHA revisado | Check local |
| --- | --- | --- |
| API | `4ff580e4c1a863695c22813a0aaafc8d01fa6465` | Restore locked, build com zero warnings e 326 testes aprovados. |
| UI | `96199fe6519be81c1dd200d5f23aeff33f6a677b` | `npm run check` com lint, build e 188 testes aprovados. |
| Testes | `ed85c3c262651606ff717a889e847586ea55d32c` | Descoberta Playwright, análise sintática Python e ensaio HTTP com PostgreSQL e Moto. |
| Infraestrutura | `422811c6d27bde6da2ee9fef1a4162c436b348fb` | Nove testes unitários de configuração e ensaio SNS/SQS com filtragem e entrega raw. |

O `git diff --check` passou nos cinco repositórios alterados. O primeiro ensaio Chromium aprovou o cenário de convites. O cenário de configuração encontrou um seletor ambíguo antes de confirmar o registro. O seletor corrigido aguarda o reteste nos SHAs integrados em `dev`.
