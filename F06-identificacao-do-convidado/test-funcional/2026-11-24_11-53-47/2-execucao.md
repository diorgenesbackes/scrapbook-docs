# Aceite local do retorno à S17

Os quatro critérios passaram no candidato integrado em `dev`. O ensaio usou dois eventos sintéticos abertos e a integração HTTP da F05 no Docker. [Ambiente e matriz de navegadores](../../test-e2e/f06-story-002.md)

| Critério | Observado | Resultado |
| --- | --- | --- |
| CA1 | Depoimento e momento reutilizaram a confirmação do mesmo evento. Outro evento abriu com nome vazio e caixa desmarcada. | Aprovado |
| CA2 | Editar nome moveu o foco para o campo preenchido e desmarcou a concordância. Continuar exigiu nova aceitação e o nome atualizado apareceu ao voltar à outra categoria. | Aprovado |
| CA3 | Remover o registro e bloquear o armazenamento produziram a orientação esperada. A nova confirmação permitiu continuar na página com o rascunho intacto. | Aprovado |
| CA4 | O fluxo recusou o registro incompleto e o registro copiado entre eventos. Os unitários verificaram também a versão do aviso e o contrato para a F08. | Aprovado no recorte |

As duas categorias foram exercitadas pelas prévias locais da S17. O aceite dos formulários reais e a confirmação de envio seguem nas histórias da F07 e F08. [Plano](1-plano.md) · [Contrato](../../dev-front/f06-story-002.md)
