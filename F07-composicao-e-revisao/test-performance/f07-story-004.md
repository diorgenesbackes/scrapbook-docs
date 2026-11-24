# Prontidão e liberação da câmera em 4G emulado

O ensaio de 24/11/2026 mediu a UI `d05fb32` e os testes `97c2e35` com Playwright 1.65.0 e Chromium 158.0.8127.18. O Apple M4 Pro tinha 12 núcleos e 24 GB. A emulação usou 393 × 851 px, RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s, sem limitação artificial de CPU. Houve cinco contextos com cache desativado e duas capturas por contexto. A câmera sintética recebeu permissão prévia para separar a espera pela pessoa da abertura do dispositivo.

| Marco | Medições em ms | Mediana | Máximo |
| --- | --- | --- | --- |
| Navegação até editor utilizável | 1.957, 1.944, 1.942, 1.943, 1.937 | 1.943 ms | 1.957 ms |
| Ação de abrir até o disparador habilitado | 210, 101, 118, 116, 111, 101, 117, 99, 110, 101 | 110,5 ms | 210 ms |
| Disparo até encerramento da trilha | 4, 3, 3, 4, 3, 5, 4, 4, 3, 3 | 3,5 ms | 5 ms |
| Disparo até prévia JPEG decodificada | 19, 13, 14, 15, 19, 16, 14, 14, 25, 14 | 14,5 ms | 25 ms |

Todas as aberturas ficaram abaixo de 3 s. As dez capturas encerraram as trilhas antes de entregar a prévia. O ensaio cobre uma pessoa por contexto com câmera sintética. Envio concorrente e transferência de mídias integram a validação da F08. [PRD](../../PRD.md) · [Contrato](../dev-front/f07-story-004.md)
