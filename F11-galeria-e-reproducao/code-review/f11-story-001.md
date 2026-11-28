# Revisão técnica da F11-STORY-001

Revisão independente de 28/11/2026 no escopo de lista e detalhe autorizados. Serviço `5d89b79`, UI `4bb3887` e suítes `7f84801`. [Cartão](https://trello.com/c/pg91oOJY) · [Especificação](../spec.md)

| Aspecto | Verificação |
| --- | --- |
| Autorização e privacidade | Consultas de vínculo e abertura antes da leitura e antes da resposta, cursor vinculado ao contexto, negação genérica e cabeçalhos sem cache. |
| Consulta e projeção | Ordem composta com posição de foto, precisão civil, busca indexável, limite de página e seleção de artefatos pela versão de perfil. |
| Interface | Suspensão síncrona da superfície protegida, cancelamento de pedidos, filtros, leitura de texto, estados de mídia e recuperação de foco. |
| Testes | 854 xUnit e 200 Vitest aprovados. Build do serviço sem avisos ou erros. Lint, TypeScript e build da UI aprovados. |

A revisão encontrou perda do filtro, página e depoimento ao retomar uma aba com acesso ainda válido. A correção desmonta o conteúdo antes de suspender, guarda apenas intenção de navegação em memória, recarrega página e detalhe após nova autorização e descarta essa intenção após bloqueio. O teste de regressão reproduziu a perda antes da correção e passou na versão corrigida. A revisão do ajuste confirmou o comportamento esperado.

A continuidade do cursor entre réplicas precisa de chaves Data Protection compartilhadas e protegidas. O laboratório usa o volume de chaves previsto no Compose. Em homologação, a configuração externa precisa ser confirmada antes de comprovar alternância entre tarefas. [Decisões](../spec.md#3-decisões-técnicas)
