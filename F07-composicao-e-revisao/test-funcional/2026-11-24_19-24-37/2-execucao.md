# Aceite local da F07-STORY-005

Em 24/11/2026, o ensaio integrado usou UI `6156565`, testes `948bc5a`, serviço `76fd9e0` e eventos sintéticos com janela servida pela F05. [Plano](1-plano.md) · [Card](https://trello.com/c/AtKDv6RF)

| Critério | Evidência | Resultado |
| --- | --- | --- |
| CA1 | O navegador solicitou dispositivos apenas ao gravar. Houve parada manual e parada automática de vídeo de momento e vídeo e áudio de depoimento. O relógio de navegador confirmou 0:01 no último segundo e parada em 60.000 ou 300.000 ms. A saída nativa foi decodificada na revisão. | Aprovado |
| CA2 | Os unitários cobriram saída vazia, tamanho e duração, resultado de sonda ausente e encerramento de trilha. A interrupção no navegador preservou o arquivo anterior e a legenda. | Aprovado |
| CA3 | O editor preservou a seleção de arquivo na recusa de dispositivo e na falta de codec. O momento não ofereceu áudio. A gravação de áudio de depoimento pediu somente microfone. | Aprovado |
| CA4 | A revisão começou com o player pausado. Refazer trocou a URL após validar a nova gravação e reteve a legenda. Permanecer manteve a tentativa ativa e descartar encerrou as trilhas. Outro evento iniciou sem o rascunho. A parada automática gerou zero escritas HTTP. | Aprovado |

O aceite local usou Chromium e Firefox com dispositivos sintéticos, incluindo Android emulado. Dispositivos físicos e seus diálogos de acesso ficam para homologação. [Navegador](../../test-e2e/f07-story-005.md)
