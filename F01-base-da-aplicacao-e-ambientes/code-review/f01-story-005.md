# F01-STORY-005 · Revisão técnica

A revisão cobre os workflows, os scripts de publicação e recuperação, o validador do RDS e o procedimento operacional. O fluxo usa `hlg` e `main`, mantém a reserva de versão por repositório e vincula a recuperação à branch, ao ambiente, à execução e ao artefato conhecidos. A etapa de build da UI usa somente a configuração pública e permissão de leitura. Os jobs de publicação têm OIDC no ambiente correspondente.

O teste de destino divergente encontrou uma condição composta que deixava a publicação da UI prosseguir no Bash 3.2 local. A validação agora encerra explicitamente o script se a branch do Amplify divergir ou se as origens forem inválidas. O reteste recusou o alvo antes da criação do deployment. A revisão também conferiu o marcador do banco, os papéis do migrador, a configuração da API e a prontidão das tarefas de produção.

| Verificação | Resultado |
| --- | --- |
| API | Restore com lockfile, build com zero warnings, verificação de formato e 45 testes unitários aprovados. |
| UI | Oxlint, TypeScript, Vite e 10 testes unitários aprovados. O npm emitiu aviso de `allowScripts` da dependência opcional `fsevents` no macOS. |
| Liquibase | 20 testes no PostgreSQL descartável aprovados, incluindo pré-validação do migrador e catálogo, rollback e reaplicação. |
| Infraestrutura e testes | Nove testes de infraestrutura, quatro do smoke HTTPS e 61 cenários de workflows simulados aprovados. |
| Workflows | Sintaxe YAML e Bash, consistência das cópias de suporte e revisão dos gatilhos e permissões. |

A revisão encerrou sem achados técnicos remanescentes no diff. O AWS CLI foi simulado na suíte dos workflows. As políticas IAM e os recursos reais exigem conferência operacional. [Procedimento de ativação](../delivery/f01-story-005-producao.md)
