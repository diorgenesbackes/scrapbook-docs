# F01-STORY-003 · Revisão técnica em 20/11/2026

## Achados

### P1 · O bootstrap indicado para o RDS falha com o administrador padrão

O roteiro de homologação passou a chamar o bootstrap da F01-STORY-001. O bootstrap cria a role proprietária e executa `CREATE SCHEMA ... AUTHORIZATION scrapbook_owner` sem garantir que o operador possa assumir essa role. A criação via `CREATEROLE` concede administração da role com `SET FALSE` por padrão. Em PostgreSQL 18.6 descartável, um dono do banco com `CREATEROLE`, `CREATEDB` e `NOSUPERUSER` recebeu `must be able to SET ROLE "scrapbook_owner"`. O schema permaneceu ausente. O controle com superusuário passou, assim como a suíte existente que usa essa identidade. O administrador padrão documentado do RDS é `NOSUPERUSER`. A operação precisa de uma concessão explícita e temporária para assumir a role proprietária, exercitada em teste com administrador equivalente ao destino.

A dependência SQL vem da F01-STORY-001. A F01-STORY-003 passou a acioná-la no RDS. [Bootstrap](../../../scrapbook-liquibase/config/bootstrap.sql#L48) · [Roteiro de homologação](../dev-infra/f01-story-003-homologacao.md) · [Administrador do RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.PostgreSQL.CommonDBATasks.Roles.rds_superuser.html) · [CREATE SCHEMA no PostgreSQL](https://www.postgresql.org/docs/18/sql-createschema.html) · [Atributos de roles do PostgreSQL](https://www.postgresql.org/docs/18/role-attributes.html)

### P1 · A reexecução parcial tenta publicar com uma versão já reservada

Nos três workflows, `publish` recebe a versão da saída de `reserve`. Ao reexecutar só os jobs que falharam, o GitHub conserva as saídas dos jobs bem-sucedidos. A tentativa muda e a versão reaparece. Se API ou Liquibase já enviou a imagem antes da falha, o job tenta empurrar novamente a tag ECR imutável. A simulação da tentativa 2 com a tag da tentativa 1 confirmou que a execução para no push. A UI também gera outro ZIP sob a mesma versão. Os três workflows reutilizam o nome do artefato de evidência. Vincule a reserva e a publicação à mesma tentativa e preserve a versão imutável e a evidência de cada execução.

O achado foi introduzido pelos workflows da F01-STORY-003. [Workflow API](../../../scrapbook-service/.github/workflows/publish-hlg.yml#L29) · [Workflow Liquibase](../../../scrapbook-liquibase/.github/workflows/publish-hlg.yml#L40) · [Workflow UI](../../../scrapbook-ui/.github/workflows/publish-hlg.yml#L40) · [Push ECR](../../../scrapbook-service/.github/scripts/release_ecs.sh#L15) · [Reexecução parcial no GitHub](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/re-run-workflows-and-jobs) · [Imutabilidade do ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)

### P2 · Timeout deixa a tarefa de migração sem reconciliação

Depois de `run-task`, o workflow espera o estado `STOPPED`. Se o waiter esgota seu orçamento ou falha, `set -e` encerra o script e o grupo de concorrência do GitHub é liberado. O waiter apenas consulta o ECS. A tarefa pode continuar e concluir a alteração depois que o job registrou falha. Outra execução ganha outro token e pode iniciar uma segunda tarefa sem conferir o estado da primeira. A simulação de exit 255 confirmou a saída após `run-task`, sem consulta final ou atualização do marcador. Trate a espera inconclusiva como estado desconhecido, preserve o ARN da tarefa e reconcilie esse estado antes de iniciar outra migração.

O achado foi introduzido pelo fluxo Fargate da F01-STORY-003. [Migração](../../../scrapbook-liquibase/.github/scripts/publish_migration.sh#L29) · [Concorrência do workflow](../../../scrapbook-liquibase/.github/workflows/publish-hlg.yml#L9) · [Waiter do ECS](https://docs.aws.amazon.com/cli/latest/reference/ecs/wait/tasks-stopped.html)

### P2 · A API ativa com um marcador de migração sem versão nem SHA

O script da API valida ambiente, geração e resultado no JSON do Parameter Store. A leitura de versão e SHA ocorre como substituição de comando dentro dos argumentos de `record`. Se `jq -er` falha, o status do comando externo ainda é o de `record`. Com os dois campos ausentes, a simulação chegou a `update-service` e registrou o texto `null` nos dois campos de proveniência. Valide a estrutura completa do marcador e interrompa a publicação se versão e SHA estiverem ausentes ou inválidos.

O achado foi introduzido pelo workflow da API. [Validação e registro do marcador](../../../scrapbook-service/.github/scripts/publish_ecs.sh#L8)

### P2 · O build da UI recebe a identidade de publicação AWS

O job assume a role AWS antes de executar o script que roda `npm ci` e `npm run build`. A Action fixada exporta por padrão as credenciais temporárias para os passos seguintes. Dependências e ferramentas da cadeia de build executam com a mesma capacidade de publicar no Amplify, apesar de precisarem apenas produzir o ZIP local. Separe o build em um job sem permissão OIDC e vincule o ZIP daquele job ao SHA e ao hash verificados na publicação.

O achado foi introduzido pelo workflow da UI. A exposição decorre do escopo e da ordem dos passos. [Workflow UI](../../../scrapbook-ui/.github/workflows/publish-hlg.yml#L64) · [Build e publicação](../../../scrapbook-ui/.github/scripts/publish_amplify.sh#L10) · [Exportação de credenciais da Action fixada](https://github.com/aws-actions/configure-aws-credentials/blob/e1253824e5c10ff9df46874f81ed3ec929e19cfd/README.md)

## Escopo

Diff de `dev...feature/F01-STORY-003` nos seis repositórios. Foram revisados código, workflows, validadores, fixtures e as dependências técnicas acionadas pelo bootstrap e pela publicação. Os efeitos de AWS foram avaliados por documentação primária, análise e simulação local.

| Repositório | Base `dev` | Revisão |
| --- | --- | --- |
| Infraestrutura | `bdf7347` | `d912641` |
| API | `550872b` | `537e725` |
| UI | `024c436` | `7dc6525` |
| Testes | `9c9a15c` | `6a05707` |
| Documentação | `f44e22a` | `fb4cbbb` |
| Liquibase | `405538d` | `3b181b1` |

O parecer segue os guidelines de engenharia do projeto. [Git Flow](../../guidelines/git-flow.md) · [Liquibase](../../guidelines/liquibase.md) · [Backend](../../guidelines/backend.md) · [Frontend](../../guidelines/frontend.md) · [Observabilidade](../../guidelines/observabilidade.md)

## Verificações técnicas

| Área | Comando ou método | Saída | Warnings e observações |
| --- | --- | --- | --- |
| API · SDK 10.0.401 | `dotnet restore ScrapBook.sln --locked-mode` | 0 | Nenhum warning. |
| API · compilação | `dotnet build ScrapBook.sln --no-restore --no-incremental --disable-build-servers` | 0 | Zero erros e warnings. |
| API · estilo | `dotnet format ScrapBook.sln --verify-no-changes --no-restore --severity info` | 0 | Relatório de diagnósticos vazio. O formatter exigiu comunicação local entre processos do MSBuild, liberada na execução verificada. |
| API · testes | `dotnet test ScrapBook.sln --no-build --no-restore` | 0 | 45 passaram. |
| UI · Node 26.13.0 e npm 11.20.0 | `npm run lint`, `npm run build` e `npm test` em execuções independentes | 0 em cada | Zero warnings do Oxlint, TypeScript e Vite. Dez testes passaram. |
| Liquibase · PostgreSQL Docker 18.6 | `python3 -B tests/test_baseline.py` | 0 | 19 passaram. A sondagem extra com administrador sem `SUPERUSER` reproduziu o P1 e o controle com superusuário passou. |
| Infraestrutura · reserva | `python3 -B -m unittest discover -s tests -v` | 0 | Quatro testes passaram. O Git emitiu quatro ocorrências de `You appear to have cloned an empty repository.` durante o preparo do remoto descartável, sem linha de código associada. O aviso faz parte da montagem dos quatro casos da fixture de reserva. |
| Testes · verificador HTTPS | `python3 -B -m unittest discover -s deploy -p 'test_*.py' -v` | 0 | Quatro passaram com respostas sintéticas. |
| Testes · workflows | `python3 -B deploy/verify_workflows.py` com os três checkouts revisados | 0 | Treze cenários passaram. Três sondagens extras reproduziram o push repetido, o marcador sem proveniência e o timeout sem reconciliação. |
| Diff e sintaxe | `git diff --check dev...HEAD`, leitura estrutural YAML e `bash -n`/`sh -n` | 0 | Nove workflows e onze scripts shell verificados. A execução no GitHub Actions e na AWS permanece sem verificação nesta revisão. |

Cobertura de linhas e branches ficou sem medição. A solução .NET e a UI carecem de coletores configurados e os testes Python foram executados pelo `unittest`. O `AGENTS.md` corrente do workspace registra ausência de limiar de cobertura. Os totais acima refletem a execução dos testes.
