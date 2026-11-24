# Prontidão da S17 em 4G emulado

Em 24/11/2026, o [candidato integrado](../test-e2e/f06-story-001.md) foi medido por Playwright e CDP em Chromium 159.0.8154.6. O host era um Apple M4 Pro com 12 núcleos e 24 GB de RAM. O navegador rodou com viewport de 393 × 851, sem limitação de CPU, RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s. Cada uma das cinco navegações usou contexto novo e cache desativado. O marco foi o campo de nome editável, a caixa visível e Continuar habilitado.

| Amostras integradas | Mediana | Máximo | Falhas |
| --- | --- | --- | --- |
| 1.764, 1.768, 1.763, 1.766 e 1.762 ms | 1.764 ms | 1.768 ms | 0 |

As amostras ficaram abaixo da referência de 3 s da página pública, usada neste ensaio local da S17. O aviso sintético é fornecido pelo frontend e não gera uma requisição separada. A história preserva o único GET da F05 e não introduz endpoint de envio. Por isso, carga concorrente de upload e latência de uma API de aviso ficam fora deste recorte. [PRD](../../PRD.md) · [Contrato](../dev-front/f06-story-001.md)
