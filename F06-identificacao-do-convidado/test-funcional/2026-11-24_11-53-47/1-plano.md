# Plano de aceite local do retorno à S17

Validar CA1–CA4 com navegador anônimo, dois eventos sintéticos abertos e relógio controlado. Usar a mesma origem em loopback para alternar as duas prévias de categoria. [F06-STORY-002](https://trello.com/c/lW6dEG0h) · [PRD](../../../PRD.md) · [S17](../../../design-system/screens/S17.md)

| Critério | Cenário e resultado esperado |
| --- | --- |
| CA1 | Confirmar a identificação em depoimento, abrir momento do mesmo evento e ver o nome lembrado. Em outro evento, encontrar nome vazio e caixa desmarcada. |
| CA2 | Selecionar Editar nome, encontrar a caixa desmarcada e alterar o nome. Continuar exige nova concordância. A próxima abertura nas duas categorias usa o nome atualizado. |
| CA3 | Remover o registro depois de abrir a tela e repetir com armazenamento bloqueado. O fluxo orienta nova identificação, confirma na página atual e conserva o rascunho em memória. |
| CA4 | Usar registros incompletos, vinculados a outro evento ou de outra versão do aviso. O fluxo oferece nova confirmação. O consumidor recebe o contrato somente após ação explícita com os dados do evento atual. |

A F07 ainda integra as entradas oficiais e a F08 valida a identificação no envio. As prévias locais exercitam a mesma etapa nos dois contextos com aviso e rascunho sintéticos. [Contrato](../../dev-front/f06-story-002.md)
