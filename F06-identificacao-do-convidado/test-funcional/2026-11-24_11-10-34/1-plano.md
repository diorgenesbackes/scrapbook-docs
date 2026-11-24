# Plano de aceite local da primeira identificação

Validar os critérios CA1–CA4 da [F06-STORY-001](https://trello.com/c/RiJ79nyD) contra o FR-007 e a S17. Usar um navegador anônimo, dois eventos sintéticos abertos com relógio controlado e um código desconhecido. Os cinco perfis Playwright usam contextos independentes.

| Critério | Cenário e resultado esperado |
| --- | --- |
| CA1 | A S17 mostra o nome do evento que a F05 resolve. O código desconhecido exibe indisponibilidade e omite o formulário. |
| CA2 | O aviso informa o acesso exclusivo dos responsáveis conforme a política do evento. A caixa começa desmarcada e o fluxo é anônimo. |
| CA3 | Nome vazio ou com espaços e concordância ausente impedem avançar. As mensagens do PRD são associadas aos controles e o primeiro inválido recebe foco. |
| CA4 | Com nome e concordância, Continuar confirma a identificação local. O consumidor recebe o nome normalizado, o código atual e a referência ao aviso. Em outro evento, a identificação começa vazia. |

O aceite do recorte usa dados e aviso sintéticos em Docker. Reutilização no navegador, integração das duas categorias e confirmação do envio são avaliadas nas entregas correspondentes. [Contrato](../../dev-front/f06-story-001.md) · [PRD](../../../PRD.md) · [S17](../../../design-system/screens/S17.md)
