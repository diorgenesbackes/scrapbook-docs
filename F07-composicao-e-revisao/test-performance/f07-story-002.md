# Prontidão e prévias de fotos em 4G emulado

O ensaio de 24/11/2026 usou `9c7aa7c` na UI, `8c8dfcf` no runner, Playwright 1.65.0 e Chromium 159.0.8154.6. O perfil simulou viewport 393 × 851, RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s, sem limitação artificial de CPU. O host Apple M4 Pro tinha 12 núcleos e 24 GB. Cada abertura usou contexto novo e cache desativado. Cada foto sintética era um PNG de 640 × 480 e 922.428 bytes.

| Cenário | Marco | Amostras em ms | Mediana | Máximo |
| --- | --- | --- | --- | --- |
| Uma foto | Seletor utilizável | 1.889, 1.866, 1.861, 1.861, 1.860 | 1.861 ms | 1.889 ms |
| Uma foto | Seleção até a prévia decodificada | 58, 80, 114, 91, 111 | 91 ms | 114 ms |
| Dez fotos | Seletor utilizável | 1.858, 1.864, 1.864, 1.860, 1.857 | 1.860 ms | 1.864 ms |
| Dez fotos | Seleção até as dez prévias decodificadas | 478, 469, 486, 485, 441 | 478 ms | 486 ms |

As dez aberturas ficaram abaixo da referência de 3 s, sem falhas. O tempo das prévias inclui a entrega automatizada dos arquivos locais ao navegador. A carga de API e o upload de arquivos no limite pertencem à F08. [PRD](../../PRD.md) · [Contrato](../dev-front/f07-story-002.md)
