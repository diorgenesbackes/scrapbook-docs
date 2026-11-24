# Prontidão e prévia de vídeo e áudio em 4G emulado

O ensaio de 24/11/2026 usou a UI `93e4b9c`, os testes `45a0ff9`, Playwright 1.65.0 e Chromium 159.0.8154.6. Cada abertura ocorreu em contexto novo, com cache desativado, viewport de 393 × 851, RTT de 150 ms, download de 1,6 Mbit/s, upload de 750 kbit/s e sem limitação artificial de CPU. O host Apple M4 Pro tinha 12 núcleos e 24 GB.

| Amostra | Marco | Cinco medições em ms | Mediana | Máximo |
| --- | --- | --- | --- | --- |
| MP4 / H.264, 285.436 bytes | Seletor utilizável | 1.943, 1.910, 1.911, 1.910, 1.910 | 1.910 ms | 1.943 ms |
| MP4 / H.264, 285.436 bytes | Metadados lidos | 23, 47, 53, 47, 47 | 47 ms | 53 ms |
| MP4 / H.264, 285.436 bytes | Prévia disponível | 47, 71, 77, 71, 71 | 71 ms | 77 ms |
| WebM / VP8, 153.030 bytes | Seletor utilizável | 1.909, 1.910, 1.920, 1.967, 1.909 | 1.910 ms | 1.967 ms |
| WebM / VP8, 153.030 bytes | Metadados lidos | 44, 49, 50, 17, 43 | 44 ms | 50 ms |
| WebM / VP8, 153.030 bytes | Prévia disponível | 68, 73, 77, 42, 66 | 68 ms | 77 ms |
| WAV / PCM, 264.678 bytes | Seletor utilizável | 1.910, 1.909, 1.909, 1.909, 1.910 | 1.909 ms | 1.910 ms |
| WAV / PCM, 264.678 bytes | Metadados lidos | 51, 49, 49, 48, 48 | 49 ms | 51 ms |
| WAV / PCM, 264.678 bytes | Prévia disponível | 76, 74, 76, 71, 75 | 75 ms | 76 ms |

As 15 aberturas ficaram abaixo da referência de 3 s. Os tempos de metadados e prévia incluem a entrega automatizada dos arquivos locais ao navegador. O ensaio mede prévias das amostras indicadas. Carga da API e envio de arquivos no limite pertencem à F08. [PRD](../../PRD.md#f07-composição-e-revisão) · [Navegador](../test-e2e/f07-story-003.md)
