# Plano de aceite da seleção de fotos

Validar CA1–CA4 em Docker com dois eventos sintéticos, janela controlada, arquivos locais e identificação da F06 com aviso sintético. [Card](https://trello.com/c/djeVwk02) · [PRD](../../../PRD.md#f07-composição-e-revisão) · [Contrato](../../dev-front/f07-story-002.md)

| Critério | Cenário e resultado esperado |
| --- | --- |
| CA1 | Evento aberto oferece ambas as entradas. Janela fechada impede ingresso. Depoimento aceita uma foto e momento aceita dez. Arquivo excedente em tamanho ou quantidade bloqueia a revisão até recuperação. |
| CA2 | JPEG e PNG admitem prévia. Arquivo vazio, assinatura incompatível e imagem corrompida têm feedback por arquivo. HEIC exibe prévia quando o navegador decodifica ou oferece recuperação conservando a seleção válida. |
| CA3 | Troca mantém a posição, remoção preserva a ordem e a revisão tem uma legenda única. O contador aparece aos 100 restantes e o limite é 500. O contrato preserva `File`, formato e bytes. |
| CA4 | Sair ou descartar exige confirmação quando há dados pendentes. Permanecer conserva o conjunto. Troca, remoção e descarte revogam URLs. Os dados ficam na página, com nenhuma transmissão durante a revisão. |

Executar a regressão de texto e da página pública, medir dez navegações móveis com cache frio e comparar a prontidão e a decodificação para uma e dez fotos.
