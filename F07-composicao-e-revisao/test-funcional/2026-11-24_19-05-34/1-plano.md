# Plano de aceite da F07-STORY-004

Validar em Docker com dois eventos sintéticos, relógio controlado, Chromium com câmera emulada e imagens próprias. A resposta HTTP e a janela vêm da stack local. [Card](https://trello.com/c/uxlzqFB7) · [PRD](../../../PRD.md#f07-composição-e-revisão) · [Contrato](../../dev-front/f07-story-004.md)

| Critério | Cenário e evidência esperada |
| --- | --- |
| CA1 | Entrar no editor sem chamadas de câmera. Abrir apenas com ação do convidado e solicitar só vídeo. Capturar no depoimento e completar dez fotos no momento. Aplicar validação de arquivo e limite de 20.000.000 bytes. |
| CA2 | Recusar câmera no navegador e manter o conjunto. Mostrar a orientação prevista e completar o conjunto com arquivo do dispositivo. |
| CA3 | Revisar o JPEG capturado junto da legenda. Refazer mantém a posição. Falha de geração do JPEG preserva a foto anterior e permite nova tentativa. Trocar e remover seguem os controles do editor compartilhado. |
| CA4 | Após uma foto, cancelamento, mudança de formato, saída e descarte, medir o encerramento das trilhas. Incluir permissão que termina tardiamente e revogação de URL na troca ou no descarte. Verificar isolamento sem envio, armazenamento de conteúdo ou registro persistido. |
