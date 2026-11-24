# Emissão e consulta do convite

A API expõe a consulta corrente e a emissão autenticadas sob `/api/v1/events/{eventId}/invitations`. O mesmo `EventController` da F03 aplica a identidade, a origem e a validação HTTP. A regra de emissão está em `InvitationService`, que trava a linha do evento antes de consultar ou modificar o convite. O schema vigente do Liquibase fornece as tabelas, chaves e índices.

Emissão concorrente com a mesma chave devolve o mesmo convite válido. Uma nova operação substitui o pendente e registra a transição e a emissão na mesma transação. Convite terminal perde o ciphertext. A consulta corrente verifica a borda de 24 horas com o relógio da API e preserva os estados neutros da trava estrutural.

A API usa chaves distintas de HMAC e AES-GCM. O Docker pode gerar e reaproveitar chaves privadas localmente. A ausência de configuração desabilita as operações de convite com `503` para um evento autorizado. [Contrato](../spec.md#5-contratos-de-api)
