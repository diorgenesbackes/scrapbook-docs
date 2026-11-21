# F01-STORY-004 · Revisão técnica

O escopo inclui os caminhos de publicação reutilizados e os novos workflows manuais da API, UI e Liquibase. Foram examinados o vínculo entre execução e artefato, os checks antes da ativação, a evidência produzida em falha, as permissões dos jobs e a restrição do rollback ao banco descartável.

## Correções verificadas

- Uma validação dentro de uma função de shell podia continuar quando o chamador usava encadeamento condicional. As verificações de origem retornam falha explicitamente. A simulação confirmou recusa de execução malsucedida e de evidência de outro repositório sem atualizar o serviço ECS.
- Uma divergência de digest ou mudança do estado observado precisa falhar antes da ativação. Os controles explícitos foram conferidos com digest ECR diferente, definição ECS divergente e job Amplify alterado.
- O pré-check de rollback exige o alvo descartável e seu único changeset esperado. A simulação confirmou recusa de instância compartilhada e URL da tarefa divergente, sem executar a tarefa.

## Verificações

| Projeto | Execução | Resultado |
| --- | --- | --- |
| Service | Restore com lockfile, build sem incremento, `dotnet format --verify-no-changes` e testes | Código 0. Sem warnings no build. 45 testes passaram. |
| UI | `npm run lint`, `npm run build` e `npm test` | Código 0 nos três comandos. Sem warnings reportados. 10 testes passaram. |
| Tests | `python3 -B -m unittest discover -s deploy` | Código 0. Quatro testes passaram. |
| Workflows | Sintaxe Bash/sh e parsing dos três YAMLs | Código 0. Validação local da sintaxe. |
| Publicação e recuperação | `deploy/verify_workflows.py` com respostas simuladas | Código 0. 27 cenários de publicação e 13 de recuperação passaram. |

A revisão não encontrou novos desvios técnicos no escopo verificado após os retestes. O GitHub hospedado, as permissões efetivas da AWS e a disponibilidade de artefatos de homologação continuam pendentes. [Procedimento e requisitos de ativação](../delivery/f01-story-004-recuperacao.md)
