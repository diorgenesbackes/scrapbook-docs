# Prontidão do editor e da revisão em 4G emulado

O ensaio em 24/11/2026 usou o frontend `43c3862` e os testes `142c9fc` em Docker. Playwright 1.65.0 e Chromium 159.0.8154.6 rodaram em Apple M4 Pro com 12 núcleos e 24 GB, sem limitação artificial de CPU. Cada navegação abriu um contexto novo, com cache desativado, viewport 393 × 851, RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s.

| Marco | Amostras em ms | Mediana | Máximo |
| --- | --- | --- | --- |
| Rota direta até campo editável | 1.828, 1.826, 1.824, 1.824, 1.823 | 1.824 ms | 1.828 ms |
| Acionar Revisar até a prévia de 5.000 caracteres | 39, 36, 40, 37, 43 | 39 ms | 43 ms |

As cinco aberturas ficaram abaixo da referência de 3 s da página pública, com zero falhas. A medição de revisão inclui a interação automatizada do navegador. O fluxo herda a consulta da F05 e processa o texto localmente. Carga de upload e latência de um endpoint de envio são verificadas na F08. [PRD](../../PRD.md) · [Contrato](../dev-front/f07-story-001.md)
