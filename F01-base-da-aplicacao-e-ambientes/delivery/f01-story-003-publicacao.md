# F01-STORY-003 · Publicação e rastreabilidade em homologação

Os workflows publicam somente após push em `hlg` nos repositórios de Liquibase, API e UI. Os principais recursos são preparados manualmente. A ativação de `main` pertence à F01-STORY-005. [História no Trello](https://trello.com/c/Ji6EyWMB) · [Preparação da homologação](../dev-infra/f01-story-003-homologacao.md)

## Configuração do GitHub e AWS

- Defina `RELEASE_SERIES` com `MAJOR.MINOR` como variável do repositório. Use regras separadas para restringir criação de tags `v*.*.*` ao ator confiável da reserva e bloquear atualização e exclusão das tags existentes, inclusive para esse ator. O job de reserva usa `contents: write` e o mesmo namespace em todas as branches do repositório. Os jobs de PR usam leitura e ações fixadas por SHA. Proteja `dev`, `hlg` e `main` conforme o Git Flow. [Git Flow](../../guidelines/git-flow.md#9-releases-e-artefatos) · [Rulesets de tags do GitHub](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- Crie o ambiente GitHub `hlg` em cada repositório de publicação, limitado à branch `hlg`. Restrinja a trust policy de cada role AWS ao provedor OIDC do GitHub, audiência `sts.amazonaws.com` e subject exato da conta, repositório e ambiente. Confira o formato efetivo do subject do repositório ao configurar o vínculo. O job de publicação tem `id-token: write` e `contents: read`. [OIDC com AWS no GitHub Actions](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)
- Limite a role de publicação da API aos recursos ECR, serviço e definição da API, leituras de target group e `ssm:GetParameter` no marcador ativo do banco. A role do Liquibase usa seu repositório ECR, família da tarefa migradora, execução no cluster de homologação, `ssm:GetParameters` e `ssm:DeleteParameter` no registro pendente e `ssm:PutParameter` nos registros ativo e pendente. Restrinja `iam:PassRole` às roles das tarefas do componente e ao serviço ECS Tasks. A role da UI só pode criar e iniciar deploy e consultar o job na branch configurada do app Amplify. A autenticação do ECR e APIs de AWS com escopo de recurso indisponível devem receber apenas as ações necessárias.
- Os repositórios usam cópias idênticas dos pequenos scripts de reserva e suporte de release, para que cada workflow execute a versão revisada no próprio commit. Ao alterá-los, sincronize as cópias a partir de `scrapbook-infrastructure/scripts/` e rode os testes da reserva. [Código da reserva](../../../scrapbook-infrastructure/scripts/reserve_release.py)

| Componente | Variáveis do ambiente GitHub `hlg` |
| --- | --- |
| Todos | `AWS_ACCOUNT_ID`, `AWS_DEPLOY_ROLE_ARN` |
| Liquibase | `ECR_REPOSITORY`, `ECS_CLUSTER`, `MIGRATION_TASK_DEFINITION`, `MIGRATION_SUBNET_IDS` (array JSON de dois IDs), `MIGRATION_GROUP_ID` |
| API | `ECR_REPOSITORY`, `ECS_CLUSTER`, `ECS_SERVICE`, `ALB_TARGET_GROUP_ARN`, `DATABASE_GENERATION` (`f01-story-001-base`), `UI_ORIGIN` |
| UI | `AMPLIFY_APP_ID`, `AMPLIFY_BRANCH` (`hlg`), `UI_ORIGIN`, `API_ORIGIN` |

## Ordem da primeira publicação

1. Registre o candidato da base, versões compatíveis e o resultado dos pré-requisitos de rede, armazenamento e identidade. Promova primeiro o Liquibase em `hlg`. Confira o exit code do contêiner, digest e geração no registro do workflow. Uma tarefa bem-sucedida atualiza `/scrapbook/hlg/liquibase/active` com versão, revisão, geração e resultado.
2. Promova a API. O workflow exige um marcador de homologação bem-sucedida com geração esperada, versão válida e SHA completo antes de ativar a tarefa. Ele confere a configuração do target group, a revisão ECS, o digest em execução e o alvo saudável. A primeira ativação sobe o serviço de zero para uma tarefa.
3. Promova a UI e guarde a evidência do job, o SHA-256 do ZIP e o artefato para recuperação. O build usa um job sem permissão OIDC. A publicação verifica o ZIP da execução antes de assumir a role do Amplify. Verifique `release.json` na origem pública HTTPS e execute o smoke manual do shell, CSP, SPA e restrição das sondas depois da ativação. [Verificação HTTPS](../../../scrapbook-tests/deploy/check_hlg.py)
4. Registre a trilha por componente com repositório, SHA, versão, digest ou ZIP, ambiente, execução, data, resultado e checks. Preserve evidências de falha parcial. Revise estado efetivo do banco, API e UI antes de reexecutar apenas o componente necessário.

## Reexecução e migração pendente

Use **Re-run all jobs** ao reexecutar o workflow do componente. O identificador da reserva inclui a tentativa do GitHub. Uma tentativa parcial que herdar a reserva anterior encerra antes de assumir credenciais AWS. Uma reexecução completa reserva outra versão para o novo build e preserva a evidência de cada tentativa. [Reexecuções no GitHub](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/re-run-workflows-and-jobs)

O Liquibase cria `/scrapbook/hlg/liquibase/pending` antes de chamar o ECS e registra o ARN retornado da tarefa. Uma espera inconclusiva é seguida de uma consulta ao estado efetivo. Uma execução posterior reconcilia a tarefa anterior quando o ARN e o digest conferem e o contêiner terminou com código zero. Somente depois de gravar o marcador ativo o workflow libera o registro pendente. Após excluí-lo, aguarda os 30 segundos exigidos pelo Parameter Store antes de reutilizar o nome. [Exclusão de parâmetros no AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/ssm/delete-parameter.html)

Uma tarefa em andamento, uma falha, um registro inválido ou uma resposta perdida de `RunTask` mantém o bloqueio. Confira o cluster e o ARN da tarefa pelo registro pendente. Se o ARN ainda estiver ausente, correlacione o `clientToken` usado em `startedBy` com a execução ECS. Resolva o estado de execução e confira o catálogo, o histórico do Liquibase e o marcador ativo antes de liberar manualmente a pendência, em uma janela sem outro workflow de migração ativo.

## Comandos de conferência

Execute o smoke com as origens e a revisão aprovadas, no repositório de testes.

```sh
python3 deploy/check_hlg.py --ui "$HLG_UI_ORIGIN" --api "$HLG_API_ORIGIN" \
    --sha "$HLG_UI_SHA" --version "$HLG_UI_VERSION"
```

Use `describe-target-health` com credencial operacional de leitura para comprovar a prontidão interna. Compare o digest retornado por `describe-tasks` com o do registro da API. Compare a geração no Parameter Store com a evidência de sucesso da execução Liquibase. O teste do WAF e a confirmação de acessos a objetos e segredos são exercícios de homologação com recursos sintéticos.
