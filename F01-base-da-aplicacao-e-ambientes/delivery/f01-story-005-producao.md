# F01-STORY-005 · Publicação da base de produção

Os workflows de publicação usam `hlg` para homologação e `main` para produção. A produção requer aceite do candidato em homologação, autorização de publicação e a conclusão do inventário e dos controles de preparação. [Card](https://trello.com/c/2iFuwYcd) · [Preparação de produção](../dev-infra/f01-story-005-producao.md) · [Fluxo de promoção](../../guidelines/git-flow.md)

## Configurar os destinos

Nos repositórios de Liquibase, API e UI, associe o environment GitHub `production` exclusivamente à branch `main`. Mantenha `hlg` exclusivo da branch `hlg`. Configure roles OIDC dedicadas para cada combinação de repositório e ambiente, com audiência `sts.amazonaws.com` e subject exato `repo:PROPRIETÁRIO/REPOSITÓRIO:environment:production` ou `:environment:hlg` conforme o destino. Restrinja as permissões aos recursos do respectivo ambiente. Use variáveis de publicação no environment correspondente, inclusive quando o nome da variável for idêntico nos dois ambientes. `RELEASE_SERIES` é compartilhada por repositório para reservar PATCH sem colisões. [OIDC no GitHub Actions](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)

| Componente | Variáveis GitHub do environment `production` |
| --- | --- |
| Todos | `AWS_ACCOUNT_ID`, `AWS_DEPLOY_ROLE_ARN` |
| Liquibase | `ECR_REPOSITORY`, `ECS_CLUSTER`, `MIGRATION_TASK_DEFINITION`, `MIGRATION_SUBNET_IDS`, `MIGRATION_GROUP_ID`, `MIGRATION_PASSWORD_SECRET_ARN`, `MIGRATION_DATABASE_URL` |
| API | `ECR_REPOSITORY`, `ECS_CLUSTER`, `ECS_SERVICE`, `ALB_TARGET_GROUP_ARN`, `DATABASE_GENERATION` (`f01-story-001-base`), `DATABASE_SECRET_ARN`, `UI_ORIGIN` |
| UI | `AMPLIFY_APP_ID`, `AMPLIFY_BRANCH` (`main`), `UI_ORIGIN`, `API_ORIGIN` |

Para produção, autorize o Liquibase a usar somente `/scrapbook/production/liquibase/active` e `/scrapbook/production/liquibase/pending` conforme as ações da [publicação de homologação](f01-story-003-publicacao.md#configuração-do-github-e-aws). Dê à API leitura somente do parâmetro ativo de produção. Use o scope ECR e ECS de cada componente e restrinja `iam:PassRole` às roles da respectiva tarefa. Dê à UI as operações de publicação e recuperação somente no app e na branch Amplify de produção. A publicação mantém evidência por tentativa, e a recuperação valida a origem dessa evidência antes de alterar o destino.

## Ativar e conferir a primeira publicação

1. Registre o aceite de homologação com o SHA de cada repositório, a geração compatível, os checks e os resultados dos ensaios de publicação e recuperação das stories 003 e 004. Confira em `main` o diff e a ancestralidade do candidato seletivo. Justifique e revalide diferenças que afetem o candidato.
2. Promova primeiro o Liquibase. Aguarde o término da tarefa e confira código zero, digest, geração, SHA e versão no marcador ativo. A pré-validação da tarefa confirma `session_user=scrapbook_migrator` e `current_user=scrapbook_owner`. Confira também o papel de execução do ECS e o estado do histórico e do catálogo antes de promover a API.
3. Promova a API. Confira no registro que o marcador aceito identifica produção e a geração prevista. Compare revisão e digest com o serviço ECS. Confirme pelo menos duas tarefas saudáveis em duas zonas, apenas os alvos esperados saudáveis na porta `8080`, sondas operacionais internas e o piso de duas tarefas na escalabilidade. Confira os alarmes e a entrega das notificações controladas.
4. Promova a UI. Valide o SHA-256 do ZIP, o job Amplify e o `/release.json` público. Execute o smoke abaixo no repositório de testes. Abra também a página inicial e uma rota interna em navegador para conferir o HTML, a navegação e a CSP. Confirme o redirecionamento de HTTP para HTTPS, a origem explícita da UI e a ausência de diagnóstico local no build publicado.
5. Execute a matriz de isolamento da [preparação](../dev-infra/f01-story-005-producao.md#comprovar-isolamento). Registre o resultado e o candidato efetivamente ativo, inclusive em caso de falha parcial. Mantenha o aceite do ambiente pendente enquanto houver critério obrigatório por verificar.

~~~sh
python3 deploy/check_hlg.py --ui "$PRODUCTION_UI_ORIGIN" --api "$PRODUCTION_API_ORIGIN" \
    --sha "$PRODUCTION_UI_SHA" --version "$PRODUCTION_UI_VERSION"
~~~

## Recuperar a base publicada

- **API e UI.** Use `recover.yml` na ref `main` e forneça ID, tentativa, SHA e versão da última publicação bem-sucedida de produção. Na API, confirme a geração do banco e informe a revisão ECS atual. Na UI, informe o último job observado do Amplify e preserve o ZIP de origem. Confira novamente alvos, digest, `release.json` e os checks do componente. No primeiro deploy, se ainda houver zero versões aprovadas, preserve a janela e trate a falha antes de admitir tráfego.
- **Liquibase.** Em caso de execução inconclusiva, preserve o marcador pendente e use o token e o ARN registrados para reconciliar a tarefa antes de reexecutar. Para incidente de dados, escolha o ponto de recuperação com o responsável, restaure o RDS em instância separada de produção e valide conectividade, geração e dados antes de redirecionar as tarefas. A reversão de schema com dados requer um plano específico compatível com o catálogo e o workflow do Liquibase. O ensaio `recover-hlg.yml` restaura apenas o cenário descartável de homologação. [Recuperação da F01-STORY-004](f01-story-004-recuperacao.md) · [Restauração para um ponto no tempo](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)

## Registrar a referência para a F02

| Registro | Identificação a confirmar após a ativação |
| --- | --- |
| Liquibase | Primeiro changeset em `db.changelog-master.yaml`, schema `scrapbook`, geração `f01-story-001-base` e roles `scrapbook_migrator` e `scrapbook_app`. Anote versão, SHA, digest, execução, tarefa e confirmação de catálogo. |
| API | `Production`, origem explícita da UI, TLS no banco e identidade local desabilitada. Anote versão, SHA, execução, digest, revisão ECS e sinais observados. |
| UI | Origem HTTPS da API de produção. Anote versão, SHA, execução, job Amplify e SHA-256 do ZIP efetivamente servido. |
| Operação | Registre instante da verificação, responsável, publicação anterior recuperável e resultado dos critérios. A F02 usa essa referência e acrescenta o contrato de autenticação previsto no PRD. |

A versão ativa de produção e as evidências remotas de topologia, isolamento e publicação dependem da primeira ativação autorizada. [Aceite da F01](../spec.md#7-estratégia-de-testes) · [Contas e sessões no PRD](../../PRD.md#f02-contas-e-sessões)
