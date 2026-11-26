# F08-STORY-005 — Depoimento em áudio do convidado

O convidado seleciona um áudio ou grava pelo microfone e revisa o conteúdo antes do envio. A seleção preserva a validação da F07 para formato, duração e tamanho, e a revisão permite uma legenda opcional. A transferência exibe progresso dos bytes e usa as mesmas credenciais da tentativa enquanto a página mantiver o rascunho em memória. [Composição e revisão](../../F07-composicao-e-revisao/dev-front/f07-story-003.md)

A S26 distingue transferência, interrupção, validação e reconciliação. O cliente consulta a tentativa e a condição do arquivo antes de retomar. A S27 aparece depois de a API retornar o recibo com o instante de confirmação. No recibo, o conteúdo e a prévia deixam a jornada. [S26](../../design-system/screens/S26.md) · [S27](../../design-system/screens/S27.md)
