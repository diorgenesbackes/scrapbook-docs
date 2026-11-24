# Plano de aceite da F07-STORY-005

Validar duas páginas de evento em Docker, com câmera e microfone sintéticos e relógio de contribuição controlado. Exercitar início e parada com gravações nativas e usar relógio de navegador para as bordas de 60 e 300 s. Medir também uma gravação de vídeo de um minuto em tempo real. [Card](https://trello.com/c/AtKDv6RF) · [PRD](../../../PRD.md#f07-composição-e-revisão) · [Contrato](../../dev-front/f07-story-005.md)

| Critério | Cenário e evidência esperada |
| --- | --- |
| CA1 | Solicitar dispositivos apenas pela ação Gravar. Validar a parada manual, o contador aos 60 e 300 s, o último segundo, a parada automática e a abertura da revisão. |
| CA2 | Validar bytes, duração, assinatura e decodificação da saída. Uma interrupção ou saída inválida oferece recuperação e conserva a mídia anterior. |
| CA3 | Com acesso recusado ou codec indisponível, escolher um arquivo válido. Momento admite vídeo e mantém áudio indisponível. |
| CA4 | Revisar mídia sem reprodução automática. Refazer preserva a legenda, descartar libera trilhas e URLs e permanecer conserva a gravação. Verificar ausência de envio e isolamento entre eventos. |
