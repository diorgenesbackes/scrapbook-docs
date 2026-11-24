# Aceite local da F07-STORY-004

Em 24/11/2026, o navegador executou os cenários de captura sobre a UI `d05fb32`, os testes `97c2e35` e o serviço `76fd9e0`, com eventos sintéticos e a janela real da API. [Plano](1-plano.md) · [Card](https://trello.com/c/uxlzqFB7)

| Critério | Evidência | Resultado |
| --- | --- | --- |
| CA1 | A entrada no editor fez zero pedidos de câmera. Fotografar pediu só vídeo. Depoimento aceitou uma captura e momento aceitou uma captura mais nove arquivos. Ao preencher a categoria, o controle de acréscimo ficou desabilitado. Os limites do seletor compartilhado foram exercitados em unitários e nas regressões de fotos. | Aprovado |
| CA2 | O navegador recusou a câmera com `NotAllowedError`. A mensagem prevista apareceu, a foto anterior permaneceu e o seletor acrescentou outra foto válida. | Aprovado |
| CA3 | O JPEG entrou na prévia e na revisão com a legenda. Refazer manteve a posição e revogou a URL anterior após a troca. Uma falha na geração do JPEG preservou a foto e a tentativa seguinte foi aceita. A remoção habilitou nova captura. | Aprovado |
| CA4 | As trilhas ficaram encerradas ao concluir, sair do formato ou abrir o diálogo de descarte ou saída. A resposta tardia após a troca de formato teve sua trilha encerrada. Descartar revogou a prévia e o segundo evento abriu com estado independente. | Aprovado |

O resultado se aplica a Chromium desktop e Android emulado com câmera sintética. O aceite com câmera física e diálogo de permissão nativo permanece para homologação. [Evidência de navegador](../../test-e2e/f07-story-004.md)
