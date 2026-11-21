# F01-STORY-003 · Rede e banco de homologação

Este roteiro inicia a T01 da F01-STORY-003. A topologia segue a spec e o plano da F01. [História no Trello](https://trello.com/c/Ji6EyWMB) · [Spec da F01](../spec.md#ambiente-local-e-aws) · [Plano da feature](../plan.md)

## Provisionamento manual

1. Confirme a conta de homologação e a região `us-east-1`. Crie uma VPC dedicada com resolução de DNS e nomes DNS habilitados. Reserve sub-redes para ALB e tarefas ECS em duas zonas de disponibilidade, com uma rota IPv4 padrão para o Internet Gateway.
2. Crie sub-redes privadas para o banco em duas zonas. Use uma tabela de rotas sem rota para Internet Gateway e desative a atribuição automática de IP público. Inclua essas sub-redes no grupo de sub-redes do RDS. A configuração inicial usa a VPC sem NAT Gateway. A T02 acrescenta o endpoint de gateway do S3 previsto na spec. [Tabelas de rotas da VPC](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html) · [RDS em VPC](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html)
3. Crie os grupos de segurança na mesma VPC e configure as regras da tabela. Na T03, associe os grupos às tarefas e ao ALB. [Segurança de rede do ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/security-network.html)
4. Crie a instância PostgreSQL privada com armazenamento criptografado, porta TCP 5432 e o grupo de segurança do RDS. Use Single-AZ para a configuração padrão. Prepare credenciais de acesso distintas para a API e o migrador na etapa de identidades e segredos. Aguarde a instância ficar disponível para coletar o inventário.

| Grupo | Entrada prevista |
| --- | --- |
| ALB | HTTPS TCP 443 aplicado na T03 |
| API | TCP 8080 com origem no grupo do ALB |
| Migrador | Nenhuma |
| RDS | TCP 5432 com origem nos grupos da API e do migrador |

## Verificação do inventário

Depois do provisionamento, use um perfil AWS de homologação com permissão de leitura. Defina `HLG_PROFILE`, `HLG_VPC_ID`, `HLG_DB_ID`, `HLG_PUBLIC_A`, `HLG_PUBLIC_B`, `HLG_DB_SUBNET_A`, `HLG_DB_SUBNET_B`, `HLG_ALB_SG`, `HLG_API_SG`, `HLG_MIGRATOR_SG` e `HLG_DB_SG` com os identificadores do ambiente. Execute no diretório `scrapbook-infrastructure`.

```sh
set -eu
aws --profile "$HLG_PROFILE" --region us-east-1 sts get-caller-identity
```

Após conferir a conta retornada, capture as respostas em um diretório local com acesso restrito.

```sh
set -eu
umask 077
inventory_dir=$(mktemp -d)
aws_hlg() {
    aws --profile "$HLG_PROFILE" --region us-east-1 --output json --no-cli-pager "$@"
}

aws_hlg ec2 describe-vpcs --vpc-ids "$HLG_VPC_ID" > "$inventory_dir/vpcs.json"
aws_hlg ec2 describe-subnets --filters "Name=vpc-id,Values=$HLG_VPC_ID" > "$inventory_dir/subnets.json"
aws_hlg ec2 describe-route-tables --filters "Name=vpc-id,Values=$HLG_VPC_ID" > "$inventory_dir/route-tables.json"
aws_hlg ec2 describe-security-groups --filters "Name=vpc-id,Values=$HLG_VPC_ID" > "$inventory_dir/security-groups.json"
aws_hlg rds describe-db-instances --db-instance-identifier "$HLG_DB_ID" > "$inventory_dir/rds.json"

python3 scripts/validate_hlg_network.py "$inventory_dir" \
    --vpc-id "$HLG_VPC_ID" --db-identifier "$HLG_DB_ID" \
    --public-subnet-ids "$HLG_PUBLIC_A" "$HLG_PUBLIC_B" \
    --database-subnet-ids "$HLG_DB_SUBNET_A" "$HLG_DB_SUBNET_B" \
    --alb-group-id "$HLG_ALB_SG" --api-group-id "$HLG_API_SG" \
    --migration-group-id "$HLG_MIGRATOR_SG" --database-group-id "$HLG_DB_SG"
```

O validador lê os arquivos localmente. Ele retorna `0` para inventário aprovado, `1` para divergências e `2` para entrada inválida. Registre a data e o resultado da execução na evidência da T01 sem incluir os arquivos brutos no Git. As associações dos grupos no ECS e a conectividade da API e do migrador com o RDS serão verificadas após a T03. [Validador](../../../scrapbook-infrastructure/scripts/validate_hlg_network.py)

## Estado da T01

O roteiro e o validador foram preparados localmente. O provisionamento e a validação com recursos reais de homologação estão pendentes.
