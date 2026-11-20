# F01-STORY-003 · Preparação manual da homologação

Use uma conta e um perfil de homologação revisados em `us-east-1`. Guarde o inventário e as referências reais em armazenamento operacional restrito. O roteiro de VPC e RDS cobre a T01. [Rede e RDS](f01-story-003-rede-rds.md)

## Armazenamento, chaves e identidades

- Crie buckets de homologação por finalidade conforme a spec. Ative o bloqueio de acesso público nas quatro opções, ownership administrado pelo bucket e criptografia SSE-KMS com chave da homologação. Ative versionamento para os originais das lembranças. Restrinja os principals e prefixes pela matriz do HLD. Crie o endpoint de gateway do S3 nas tabelas de rotas das tarefas e restrinja a política aos buckets necessários.
- Crie segredos separados para a API e o migrador. Prepare o banco novo com o bootstrap administrativo do Liquibase em uma conexão operada dentro da VPC. Defina as senhas de cada role no canal privado. Autorize as roles de execução de tarefas a ler exclusivamente os segredos usados por seus contêineres. Limite a role de runtime da API às permissões técnicas da fundação.
- Crie repositórios ECR distintos para API e Liquibase com `IMMUTABLE`. Preserve os digests dos candidatos em uso e a versão conhecida de recuperação ao definir a política de retenção. [Estrutura do Liquibase](../../../scrapbook-liquibase/README.md) · [Imutabilidade no ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)

## ECS, RDS e prontidão

- Crie o cluster Fargate na VPC da T01. Prepare definições de tarefa Linux x86-64 com rede `awsvpc`, roles de execução e runtime separadas, e os contêineres essenciais `api` e `liquibase`. O deploy cria revisões com o digest candidato a partir dessas definições. Para a inicialização, use uma imagem de fundação conhecida por digest nas definições e mantenha o serviço da API com `desiredCount=0` até a primeira aplicação do banco.
- Na API, configure `ASPNETCORE_ENVIRONMENT=Homologation`, `DevelopmentIdentity__Enabled=false`, `Hosting__PublicUiOrigin` com a origem exata da UI e `Database__RequireTls=true`. Injete `ConnectionStrings__ScrapBook` por referência ao Secrets Manager com a role `scrapbook_app`, `SSL Mode=VerifyFull` e `Root Certificate=/app/certs/rds-us-east-1.pem`.
- Na tarefa Liquibase, injete apenas a senha do migrador por `LIQUIBASE_COMMAND_PASSWORD` e configure `LIQUIBASE_COMMAND_URL` com o hostname privado do RDS. Use a forma `jdbc:postgresql://HOST:5432/BASE?sslmode=verify-full&sslrootcert=/opt/scrapbook/certs/rds-us-east-1.pem`. A imagem aplica o changelog pelo próprio entrypoint, sem substituição de comando. Mantenha as regras de entrada do migrador vazias e reserve o grupo de banco para os dois grupos consumidores aprovados.
- Configure ALB internet-facing nas sub-redes públicas, certificado ACM regional e redirecionamento HTTP para HTTPS. Use target group IP na porta 8080, sonda `/health/ready` na porta de tráfego e matcher `200`. Atribua às tarefas ECS IP público para saída conforme a topologia sem NAT, com entrada da API apenas do grupo do ALB. Adicione antes do encaminhamento público uma regra para `/health` e `/health/*` retornar 404. A sonda do target group chega diretamente à tarefa. [Sondas do ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)

## Amplify, DNS e proteção de entrada

- Crie o app Amplify de deploy manual, com branch `hlg`, e configure o domínio HTTPS da UI. Vincule o domínio HTTPS da API ao ALB. Registre as duas origens exatas nas variáveis do workflow e na tarefa.
- Cadastre a regra de fallback de SPA do repositório de infraestrutura. Ela preserva arquivos estáticos como `release.json`. O ZIP publicado leva `customHttp.yml` com CSP configurada para a origem HTTPS exata da API. [Fallback versionado](../../../scrapbook-infrastructure/config/hlg/amplify-rewrites.json) · [Cabeçalhos para deploy manual](https://docs.aws.amazon.com/amplify/latest/userguide/setting-custom-headers.html)
- Associe um Web ACL regional ao ALB. Comece a regra de volume por IP em `Count` e preserve as métricas agregadas. Exercite o perfil do projeto incluindo concorrência atrás do mesmo IP autorizado, compare respostas 2xx, 4xx e contagem do WAF, e registre limite, janela e efeito antes de ativar `Block`. A limitação específica de tentativas de login pertence à F02. [Regras de taxa do WAF](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html)

## Logs, sinais e correlação

- Prepare grupos CloudWatch Logs restritos para API e migração em `us-east-1`, com retenção de 180 dias. Referencie-os nas definições de tarefa usando `awslogs` e prefixos distintos. A aplicação envia logs JSON por rota nominal com correlação, trace, status e duração. Os logs de migração classificam a etapa e o workflow registra a revisão, o resultado e a geração do catálogo.
- Gere o painel de API com o comando abaixo e importe o JSON em CloudWatch. Prepare alarmes de ausência de alvos saudáveis, 5xx, latência e falha de publicação. Verifique a entrega ao operador no ensaio de homologação. Use a versão da publicação e a correlação dos logs ao investigar falhas. [Guideline de observabilidade](../../guidelines/observabilidade.md)

```sh
python3 scripts/build_hlg_dashboard.py --load-balancer "$HLG_ALB_DIMENSION" \
    --target-group "$HLG_TARGET_DIMENSION" --cluster "$HLG_CLUSTER" --service "$HLG_SERVICE"
```
