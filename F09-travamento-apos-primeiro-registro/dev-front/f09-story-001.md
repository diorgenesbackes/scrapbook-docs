# F09-STORY-001 — Estado de trava na interface

Os formulários de informações, abertura e convite usam os contratos de consulta existentes. Quando a API informa a trava, as ações de edição estrutural ou convite deixam de aparecer e a mensagem permanece neutra. O link público de contribuição fica disponível conforme a regra já vigente. [S11](../../design-system/screens/S11.md) · [S13](../../design-system/screens/S13.md) · [S14](../../design-system/screens/S14.md)

As telas consultam novamente na retomada da página e após recusas aplicáveis. Se só a versão geral avançou, a abertura usa a versão atual e preserva o rascunho enquanto modo e data persistidos continuarem correspondendo à sua base. Se a política também mudou, usa a reconciliação existente e exige novo envio consciente. [Spec](../spec.md#fluxos-de-interface)
