# F01-STORY-005 · Validação funcional local

A versão de `dev` foi exercitada com a suíte de workflows, a validação de inventário e a fundação em Docker Compose. A suíte de workflows aprovou 27 casos de publicação e 13 de recuperação de homologação, 20 de produção e a recusa de publicação a partir de `dev`. As chamadas AWS CLI e GitHub CLI dos scripts receberam respostas simuladas.

Os testes usaram API `af71e5d`, UI `bcae990`, Liquibase `dec0142`, infraestrutura `bd0b0e7` e testes `d0bc6f2`.

| Critério | Resultado local | Evidência pendente no alvo |
| --- | --- | --- |
| CA1 | O validador recusou RDS fora de Multi-AZ, retenção divergente e ausência de ponto de restauração. A publicação simulada exigiu tarefas saudáveis em duas zonas. | Inventário AWS de produção, capacidade e política de escalabilidade. |
| CA2 | Os scripts recusaram o marcador, o artefato e a referência de segredo de outro ambiente. A API recusou startup com identidade local habilitada em `Production`. | Recusas efetivas de acesso entre ambientes com identidades, segredos, banco e objetos sintéticos autorizados. |
| CA3 | A configuração vinculou `hlg` a `hlg` e `main` a `production`. A API simulada exigiu banco preparado antes de iniciar tarefas. A regressão local confirmou API, UI, prontidão e recuperação. | Candidato aceito em homologação, publicação ordenada em produção e smoke HTTPS. |
| CA4 | Foram preparados o roteiro de recuperação, a recuperação simulada e o registro de referência para a F02. | Versões, digests e sinais efetivamente ativos em produção. |

[Revisão técnica](../code-review/f01-story-005.md) · [E2E local](../test-e2e/f01-story-005.md) · [Roteiro e referência da F02](../delivery/f01-story-005-producao.md)
