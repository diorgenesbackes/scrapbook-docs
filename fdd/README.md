# FDDs do Scrap Book

Este diretório reúne as especificações técnicas de funcionalidades, seguindo o [prompt de FDD](../prompts/FDD.md), o [PRD](../PRD.md), o [HLD](../HLD.md) e os [guidelines](../guidelines/README.md).

| Funcionalidade | Versão / status | Escopo | Planejamento |
| --- | --- | --- | --- |
| [Gestão de Eventos](FDD-Gestao-de-Eventos/FDD-Gestao-de-Eventos.md) | 0.2, implementada/validada localmente na feature; sem deploy | Criar/listar/consultar/editar, abertura, identidade local e capa padrão. | [112 tasks](FDD-Gestao-de-Eventos/1-planejamento.md), [testes](FDD-Gestao-de-Eventos/3-testes.md), [operação](FDD-Gestao-de-Eventos/4-operacao.md), [relatório](FDD-Gestao-de-Eventos/5-relatorio.md) |

Cada funcionalidade usa `FDD-<Funcionalidade>/FDD-<Funcionalidade>.md`. Gestão de Eventos possui planejamento, desenvolvimento, matriz de testes, operação e relatório por task. Os ciclos preservam falhas/retestes e distinguem implementação, validação local e publicação.

O quadro [Scrap Book](https://trello.com/b/6QX5TIHS/scrap-book) é o destino exclusivo do planejamento e acompanhamento no Trello. A story contém checklists com links para os cartões individuais; o status atual é mantido no quadro.

Os critérios de aceite descritos nos FDDs são planos até haver execução identificável. Quando existirem ciclos em `scrapbook-docs/testing/`, vinculá-los à funcionalidade sem duplicar resultados.

Autenticação real foi adiada por orientação do responsável. A simulação de identidade proposta para Gestão de Eventos fica restrita ao desenvolvimento local e não substitui a futura integração do Cognito.
