# Retorno e recuperação da S17 em 4G emulado

Os ensaios do [candidato integrado](../test-e2e/f06-story-002.md) mediram a tela em Chromium 159.0.8154.6 com cache desativado, viewport de 393 × 851, RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s. Cada amostra abriu um novo contexto em um Apple M4 Pro com 12 núcleos e 24 GB, sem limitação artificial de CPU. Em retorno, o marco foi o nome lembrado visível e as ações habilitadas. Nos demais estados, o marco foi o formulário editável com caixa de concordância e botão habilitado.

| Estado | Amostras em ms | Mediana | Máximo |
| --- | --- | --- | --- |
| Primeira identificação | 1.768, 1.767, 1.764, 1.761, 1.767 | 1.767 ms | 1.768 ms |
| Retorno | 1.762, 1.765, 1.767, 1.769, 1.765 | 1.765 ms | 1.769 ms |
| Armazenamento indisponível | 1.767, 1.766, 1.770, 1.767, 1.767 | 1.767 ms | 1.770 ms |

As 15 amostras ficaram abaixo da referência de 3 s, com zero falhas neste perfil. As operações adicionadas executam no navegador e preservam o GET da F05. Teste de carga concorrente de upload e latência de API de envio pertencem à etapa da F08. [PRD](../../PRD.md) · [Contrato](../dev-front/f06-story-002.md)
