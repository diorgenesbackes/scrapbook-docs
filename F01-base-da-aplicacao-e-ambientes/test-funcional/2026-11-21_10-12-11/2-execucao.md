# F01-STORY-004 · Resultado da validação local

As verificações aplicáveis ao desenvolvimento local passaram nos commits de `dev` registrados no plano. O ensaio do Liquibase usou uma imagem amd64, PostgreSQL 18.6 descartável, TLS com CA sintética e identidade de migração dedicada. [Plano](1-plano.md) · [Resultado da regressão E2E](../../test-e2e/f01-story-004.md)

| Critério | Resultado observado | Classificação |
| --- | --- | --- |
| CA1 · Reativar artefato conhecido | `deploy/verify_workflows.py` confirmou seleção e reaproveitamento do digest e do ZIP em simulação, identificação da versão ativa e recusa de artefato divergente. | Aprovado no alcance da simulação local. |
| CA2 · Reversão controlada | O entrypoint real preservou os estados durante a prévia e recusou alvo divergente e banco com dados. Reversão e reinstalação seguiram `24 → 0 → 24` tabelas e `1 → 0 → 1` entradas no histórico. Os usuários padrão das imagens leram os bundles RDS embutidos. | Aprovado no PostgreSQL descartável. |
| CA3 · Falha e recuperação parcial | Os cenários simulados confirmaram recusa quando origem, digest, geração, alvo ou estado da ativação divergiram. Na reversão real, a vitalidade permaneceu `200` e a prontidão seguiu `200 → 503 → 200`. O E2E também verificou persistência após indisponibilidade. | Aprovado no alcance integrado local. |
| Homologação | Ativação dos workflows no GitHub, IAM efetivo, recuperação no ECS e Amplify e verificação do alvo AWS. | A verificar pela pessoa responsável, a partir de `hlg`. |

O verificador dos workflows passou em 27 cenários de publicação e 13 de recuperação, com código 0. O ensaio Docker/TLS retornou código 0 e limpou os recursos descartáveis. [Procedimento e ordem da recuperação](../../delivery/f01-story-004-recuperacao.md)
