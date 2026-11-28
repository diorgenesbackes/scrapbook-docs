# Controles de execução e sinais da F10-STORY-005

A configuração inicial usa `MediaProcessing__Enabled=false` e `MediaProcessing__Paused=false`. Em um serviço já habilitado, a pausa bloqueia novas publicações de eventos de mídia da Outbox, admissão de notificações e aquisições de trabalho de foto. Aplicações manuais de histórico e redrive também são bloqueadas. Simulações continuam disponíveis. Os workers deixam de agendar novas submissões de áudio e vídeo ao provedor e continuam a processar resultados e reconciliar as intenções já registradas.

Os intervalos dos workers usam etapa, tipo de mídia, coorte e resultado de conjuntos fixos. O snapshot persistido mede Outbox, novas aquisições e reconciliação, incluindo idade do trabalho aberto mais antigo. O relógio da amostragem define a validade dos leases; trabalhos antigos sem lease vigente ficam em espera. Os histogramas observam efeitos já concluídos pela aplicação. O aceite de 300 s usa a transação de disponibilidade observada no banco e a coorte completa de confirmações.

O contrato HTTP da F08 mantém intactos os campos dos recibos e seus efeitos canônicos. O runbook descreve a ativação, a coleta dos sinais, a drenagem em pausa e o rollback. [Especificação](../spec.md) · [Procedimento operacional](../dev-infra/f10-story-005.md) · [Revisão técnica](../code-review/f10-story-005.md)
