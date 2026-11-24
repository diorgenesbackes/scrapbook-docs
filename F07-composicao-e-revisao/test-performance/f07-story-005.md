# Parada e prontidão da gravação em 4G emulado

O ensaio de 24/11/2026 usou UI `6156565`, testes `948bc5a`, Playwright 1.65.0 e Chromium 158.0.8127.18 em perfil Android. O Apple M4 Pro tinha 12 núcleos e 24 GB. A emulação usou RTT de 150 ms, download de 1,6 Mbit/s, upload de 750 kbit/s e cache desativado, sem limitação artificial de CPU.

| Tentativa | Tempo até pedir parada | Trilhas encerradas após a parada | Prévia reconhecida após a parada | Duração e tamanho do arquivo |
| --- | --- | --- | --- | --- |
| Automática no limite | 60.001 ms | 1 ms | 69 ms | 59,986167 s e 3.857.378 bytes |
| Refazer e parar manualmente | 392 ms | 1 ms | 41 ms | 0,383417 s e 19.444 bytes |
| Refazer e parar manualmente | 394 ms | 1 ms | 38 ms | 0,383417 s e 19.428 bytes |

O editor ficou utilizável em 1.987 ms. O pedido de parada automática ocorreu 1 ms depois do prazo de 60 s. Ao final de cada tentativa, todas as trilhas observadas estavam encerradas. Os tempos da prévia incluem a observação pelo Playwright. A amostra cobre uma sessão sequencial e dispositivos sintéticos. As bordas de 300 s foram exercitadas com relógio controlado. A carga de upload fica na F08. [PRD](../../PRD.md) · [Contrato](../dev-front/f07-story-005.md)
