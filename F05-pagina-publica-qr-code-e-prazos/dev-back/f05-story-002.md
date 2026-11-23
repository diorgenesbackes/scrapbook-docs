# Consulta do código de compartilhamento

`GET /api/v1/events/{eventId}/public-link` usa a identidade autenticada e a verificação de vínculo da F03. Os dois responsáveis recebem o mesmo `{ "code": "…" }` com `Cache-Control: no-store`. O código já persistido permanece válido depois de uma edição. A conta sem vínculo recebe o mesmo `404 RESOURCE_NOT_FOUND` de um evento inexistente. Uma sessão sem autenticação válida recebe `401 IDENTITY_AUTHENTICATION_REQUIRED`.

A consulta propaga o cancelamento do cliente e usa um orçamento próprio de dois segundos. O limite do servidor segue o tratamento de `503` da F05. O contrato público e o banco permanecem os da primeira story. [Contrato e prazos](../spec.md) · [Card](https://trello.com/c/ZbToV6J7)
