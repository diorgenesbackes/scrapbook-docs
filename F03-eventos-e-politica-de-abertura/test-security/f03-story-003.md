# F03-STORY-003 · Verificação defensiva local

A suíte usou três contas sintéticas, serviços em loopback e PostgreSQL descartável. As duas contas vinculadas gravaram detalhes com versões vigentes. Para a conta sem vínculo, a edição de eventos existentes, travados ou inexistentes devolveu `404 RESOURCE_NOT_FOUND`, sem dados do evento ou do registro. A resposta apresentou `Cache-Control: no-store`.

Uma solicitação anônima retornou `401 IDENTITY_AUTHENTICATION_REQUIRED`. Uma origem fora da permitida retornou `403 REQUEST_ORIGIN_REJECTED`. Uma solicitação com uma versão anterior retornou `409 EVENT_VERSION_CONFLICT`. Uma conta vinculada recebeu `409 EVENT_LOCKED` ao editar a fixture travada. Nenhum dos dois conflitos produziu gravação adicional.

O escopo foi autorização, origem, concorrência, exposição de dados e integridade no ambiente local. Identidade de produção e AWS ficam para as verificações da versão publicada.

[Contrato da API](../spec.md#5-contratos-de-api) · [Aceite](../test-funcional/f03-story-003.md)
