# FDDs do Scrap Book

Este diretório reúne as especificações técnicas de funcionalidades, seguindo o [prompt de FDD](../prompts/FDD.md), o [PRD](../PRD.md), o [HLD](../HLD.md) e os [guidelines](../guidelines/README.md).

| Funcionalidade | Versão / status | Escopo | Planejamento |
| --- | --- | --- | --- |
| [Gestão de Eventos](FDD-Gestao-de-Eventos/FDD-Gestao-de-Eventos.md) | 0.1, proposta de 2026-11-12 | Criar, listar, consultar e editar eventos; configurar abertura; identidade simulada local; capa padrão. | [Story e 112 tasks](FDD-Gestao-de-Eventos/1-planejamento.md) |

Cada funcionalidade usa `FDD-<Funcionalidade>/FDD-<Funcionalidade>.md`. O planejamento de Gestão de Eventos já está registrado; `2-desenvolvimento.md`, `3-testes.md` e `4-operacao.md` serão criados conforme o trabalho avançar, preservando datas, decisões e evidências reais.

O quadro [Scrap Book](https://trello.com/b/6QX5TIHS/scrap-book) é o destino exclusivo do planejamento e acompanhamento no Trello. A story contém checklists com links para os cartões individuais; o status atual é mantido no quadro.

Os critérios de aceite descritos nos FDDs são planos até haver execução identificável. Quando existirem ciclos em `scrapbook-docs/testing/`, vinculá-los à funcionalidade sem duplicar resultados.

Autenticação real foi adiada por orientação do responsável. A simulação de identidade proposta para Gestão de Eventos fica restrita ao desenvolvimento local e não substitui a futura integração do Cognito.
