# F01-STORY-004 · Recuperação da publicação em homologação

Os workflows manuais recuperam artefatos conhecidos da API e da UI. O workflow do Liquibase ensaia a reversão da instalação inicial em uma instância PostgreSQL descartável. A promoção ou publicação do candidato depende das evidências do ambiente real. [História no Trello](https://trello.com/c/eVGjsc6t) · [Publicação de referência](f01-story-003-publicacao.md) · [Spec da F01](../spec.md#3-decisões-técnicas)

## Diagnosticar uma falha parcial

Antes de acionar um workflow, preserve o artefato de evidência da tentativa que falhou e o da última publicação bem-sucedida de cada repositório. Registre SHA, versão, execução e tentativa. Uma execução bem-sucedida anterior identifica o candidato a recuperar. Consulte também o que está ativo no momento do incidente.

| Componente | Estado observado | Evidência do candidato conhecido |
| --- | --- | --- |
| Liquibase | Marcador ativo no Parameter Store, tarefa pendente quando houver e geração, histórico e assinatura do catálogo verificados no alvo. | Geração, versão, SHA, digest, tarefa concluída e registro de migração. |
| API | Definição do serviço ECS, revisão e digest das tarefas em execução e saúde dos alvos da porta 8080 no ALB. | Versão ECR imutável e digest da publicação bem-sucedida. Compatibilidade com a geração ativa do banco. |
| UI | Job mais recente na branch Amplify e revisão e versão efetivamente servidas em `/release.json`. | ZIP retido, SHA-256, origem da API, revisão da UI e tentativa originária. |

1. Se a migração ficou pendente, confira a tarefa registrada e o catálogo antes de voltar a publicar. Preserve o marcador da última migração confirmada e mantenha a API atual se ela ainda estiver pronta.
2. Se o banco está íntegro e compatível, recupere primeiro a API quando ela tiver falhado. Confira as tarefas e o ALB. Depois recupere a UI, se necessário, e confira shell, rota interna e versão efetiva.
3. Se apenas a UI falhou, confirme a prontidão da API e a origem que o ZIP conhecido espera. Recupere somente a UI.
4. Se o banco compartilhado exigir recuperação, preserve os dados e identifique o procedimento de restauração e a geração compatível antes de reabrir tráfego. O ensaio desta história atua somente no banco descartável. Se uma operação remota ficar inconclusiva, use a tarefa ou o job registrado para descobrir o estado efetivo antes de tentar novamente.

## Acionar as recuperações

Nos repositórios de API e UI, selecione `recover-hlg.yml` com ref `hlg`. Forneça o ID e a tentativa da publicação conhecida, seu SHA e sua versão. Na API, informe o ARN da definição observada no serviço. Na UI, informe o ID do job mais recente observado no Amplify, inclusive quando ele falhou. Preserve o workflow de recuperação e seu `release-evidence.json` com o registro do incidente.

- **API.** O workflow confere a execução de origem, a geração ativa e o digest da tag no ECR. Registra uma revisão com a configuração atual do serviço e a imagem conhecida. Confirma a definição em execução, o digest das tarefas e a prontidão de seus IPs privados no ALB.
- **UI.** O workflow recupera o ZIP da tentativa conhecida e confere SHA-256, `release.json` e origens de UI/API. Publica o ZIP preservado e confirma a revisão servida. Se o ZIP ou a evidência tiver expirado, selecione outra publicação conhecida que ainda tenha ambos.
- **Liquibase.** Execute primeiro `preview-empty-base` no workflow do banco. A prévia confere o alvo, a assinatura da geração, o único changeset ativo e a ausência de lock. Em seguida, acione `rollback-empty-base` sobre a mesma tarefa dedicada. Após a execução, confira o histórico vazio e a preservação do schema e dos controles. [Ensaio do banco](../dev-db/f01-story-004.md)

## Preparar as permissões e os alvos

GitHub Actions exige que o arquivo de um workflow manual esteja presente na branch padrão para aceitar `workflow_dispatch`. Após disponibilizar os arquivos pelo fluxo autorizado do repositório, acione somente a ref `hlg`. Os jobs também verificam a ref atual. [GitHub Actions](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#workflow_dispatch)

- API e UI usam as identidades OIDC e variáveis de homologação da publicação. A API também precisa consultar o digest da tag no ECR. A UI precisa da leitura de `amplify:ListJobs` na própria aplicação e branch.
- Liquibase usa `AWS_RECOVERY_ROLE_ARN`, `RECOVERY_CLUSTER`, `RECOVERY_TASK_DEFINITION`, `RECOVERY_RDS_IDENTIFIER`, `RECOVERY_SUBNET_IDS` e `RECOVERY_GROUP_ID` no ambiente `hlg`. Use instância RDS privada e descartável marcada com `Environment=hlg` e `Purpose=scrapbook-recovery-rehearsal`, banco `scrapbook_recovery`, subsistema isolado da API e credencial própria da tarefa. A definição deve apontar para o digest de uma imagem que contenha os comandos desta história. A role pode consultar a instância e executar e consultar somente a tarefa dedicada no cluster autorizado. O ensaio não escreve no marcador da migração ativa.
