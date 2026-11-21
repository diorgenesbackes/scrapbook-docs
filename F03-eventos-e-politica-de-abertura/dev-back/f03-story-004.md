# F03-STORY-004 · Desenvolvimento de backend

`GET /api/v1/events/{eventId}/opening-status` usa o vínculo atual em eventos e associações antes de decidir o estado com o relógio injetado. A mesma decisão de acesso é usada no detalhe. A abertura ocorre no primeiro instante válido do dia civil em Brasília e independe do envio de avisos. A resposta é protegida por `no-store` e carrega somente estado e datas.

A fronteira da F11 é a autorização da conta vinculada e da política vigente antes de qualquer acesso ao conteúdo. `EventService.GetOpeningStatusAsync` concentra a consulta, e `EventOpeningService.DescribeContent` concentra a regra temporal para a camada Application. A futura galeria deve aplicar a decisão no próprio caminho de leitura ou emissão e dentro do mesmo escopo de consistência. O schema do Liquibase já fornece o estado necessário.

[Contrato de estado e autorização](../spec.md#estado-de-abertura-e-autorização-da-galeria)
