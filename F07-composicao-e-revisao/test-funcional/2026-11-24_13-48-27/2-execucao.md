# Aceite local do depoimento em texto

O candidato integrado em `dev` passou nos critérios de composição e revisão. O runner saiu com código 0. [Ambiente e cenários](../../test-e2e/f07-story-001.md)

| Critério | Observado | Resultado |
| --- | --- | --- |
| CA1 | Link aberto com o código correto, HTTP real da F05 no ingresso direto, janela recusada antes ou no fechamento e evento indisponível para código desconhecido. | Aprovado |
| CA2 | Orientação para texto vazio, contador a 500 restantes, truncamento de colagem Unicode com preservação do sufixo e edição posterior. | Aprovado |
| CA3 | O texto se manteve durante a identificação. Os unitários confirmaram o contrato de código, categoria, formato e identificação do evento corrente. | Aprovado no recorte local |
| CA4 | Voltar permitiu revisar a edição. A saída aguardou decisão, devolveu foco ao permanecer e descartou somente após confirmação. | Aprovado |
| CA5 | O fechamento manteve o texto e bloqueou a próxima etapa. O navegador não fez requisição de envio e o banco manteve zero contribuições. | Aprovado |

A revisão usa aviso sintético em loopback. A F08 integrará a validação definitiva, o envio e a confirmação. [Plano](1-plano.md) · [Contrato](../../dev-front/f07-story-001.md)
