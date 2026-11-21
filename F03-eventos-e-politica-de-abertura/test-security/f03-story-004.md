# F03-STORY-004 · Verificação defensiva local

A consulta retorna `401 IDENTITY_AUTHENTICATION_REQUIRED` sem identidade. Uma terceira conta recebeu `404 RESOURCE_NOT_FOUND` para eventos existentes e inexistentes. Dois responsáveis vinculados receberam o mesmo estado. A exclusão da associação no laboratório tornou a próxima consulta indisponível para a conta removida.

Mesmo com um registro confirmado sintético no evento fechado, a resposta 200 continha somente `contentState`, `opensOn`, `opensAt` e `serverTime`. Foi observado `Cache-Control: no-store`. A área fechada exibiu data e contador. O backend consulta eventos e vínculos para resolver o estado, e a fronteira para consultas de conteúdo e URLs está definida no contrato da F11.

[Contrato](../spec.md#estado-de-abertura-e-autorização-da-galeria) · [Aceite](../test-funcional/f03-story-004.md)
