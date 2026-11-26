# Frontend da F08-STORY-003

O envio usa as fotos na ordem da revisão e uma legenda comum. O progresso agrega os bytes do PUT atual e das fotos já transferidas ou reconhecidas. A retomada consulta as posições pendentes usando a chave e a credencial originais. O estado do servidor distingue transferência, reconciliação após o prazo, validação e confirmação.

Após rejeição, a revisão exibe o motivo junto da foto correspondente. O resultado por expiração indica as posições cujo recebimento ficou pendente. O recibo genérico só aparece após a confirmação do servidor. O lint e o build passaram. Os 135 testes passaram, incluindo os cenários de retomada e a regressão de foto única. [Especificação](../spec.md) · [Verificação local](../test-funcional/2026-11-25_20-40-42/2-execucao.md)
