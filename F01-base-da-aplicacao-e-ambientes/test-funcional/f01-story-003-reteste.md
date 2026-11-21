# F01-STORY-003 · Reteste das correções em 20/11/2026

| Achado | Evidência do reteste |
| --- | --- |
| Bootstrap com administrador sem SUPERUSER | Os bancos descartáveis da suíte foram preparados com um dono `NOSUPERUSER` e `CREATEROLE`. O bootstrap passou na instalação e na reaplicação. Um caso verificou que a concessão temporária é retirada e que uma associação já existente conserva as opções originais. |
| Reserva vinculada à tentativa | Cinco testes Git passaram. A saída do job associou a versão à tentativa. A nova tentativa da API e da UI com a reserva anterior foi recusada antes de chamadas externas. Os três workflows usam a mesma verificação. |
| Migração pendente após timeout | A tarefa ainda em andamento reteve o marcador e bloqueou outra migração. Uma tarefa concluída com sucesso pôde ser reconciliada. Falha de tarefa, ARN ausente, falha ao gravar o marcador ativo e disputa pela reserva mantiveram a proteção. |
| Marcador de migração exigido pela API | Marcadores sem versão ou SHA, com formato inválido ou de geração divergente foram rejeitados antes de `update-service`. O marcador válido permitiu o fluxo simulado. |
| ZIP gerado sem credencial de publicação | O build real da UI produziu o ZIP sem credenciais AWS. As simulações rejeitaram conteúdo alterado, revisão divergente e alteração da origem configurada antes do deploy. A validação estática confirmou que o job de build tem leitura no GitHub e que o job de publicação verifica o artefato antes de assumir a role. |

Passaram 27 cenários simulados dos workflows, 20 testes do Liquibase, 45 da API, 10 da UI e quatro do verificador HTTPS. Lint e build da UI passaram. Foram conferidos nove workflows e treze scripts shell. Durante a instalação local do npm, o aviso de `allowScripts` para a dependência opcional `fsevents@2.3.3` foi emitido. O Git também avisou que os cinco repositórios de teste da reserva estavam vazios na clonagem inicial.

A execução no GitHub Actions e em AWS permanece pendente. Os testes locais usaram PostgreSQL descartável e respostas simuladas para a publicação. [Parecer da revisão](../code-review/f01-story-003.md) · [Roteiro de publicação e reconciliação](../delivery/f01-story-003-publicacao.md)

## Reteste da regressão no workflow da UI

O passo inicial da publicação deriva `SITE_ARCHIVE` de `RUNNER_TEMP` e grava o caminho em `GITHUB_ENV` depois de validar a reserva. A verificação estática encontrou zero contextos inválidos no `env` de job dos nove workflows. Em processos separados, o passo inicial exportou o mesmo caminho usado pelo download, pela verificação e pelo upload simulado, inclusive com espaços no diretório. Uma reserva de outra tentativa encerrou o passo inicial antes da exportação. Os 27 cenários simulados passaram novamente. [Segunda revisão](../code-review/f01-story-003-revisao-02.md) · [Variáveis entre passos no GitHub Actions](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-commands#setting-an-environment-variable)
