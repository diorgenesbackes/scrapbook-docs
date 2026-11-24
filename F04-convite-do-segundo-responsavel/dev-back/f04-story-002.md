# Prévia e aceite do convite

As operações autenticadas `POST /api/v1/invitations/preview` e `POST /api/v1/invitations/accept` recebem o token em JSON, validam a origem e devolvem `no-store`. A prévia devolve o nome do evento e a validade para uma conta ativa e confirmada. Quando a mesma conta já aceitou e mantém o vínculo, devolve somente o identificador e o estado aceito.

A transação estabiliza a conta com lock compartilhado. Depois de localizar o hash, bloqueia o evento e o convite nessa ordem, avalia a janela e lê os vínculos atuais. No aceite, grava o slot 2, consome o convite, remove o segredo recuperável e escreve a auditoria. Um replay autorizado recupera o resultado sem outra escrita. A base Liquibase já fornece as colunas e restrições.

O codec encontra o token pela chave HMAC indicada no próprio token. Para manter convites vigentes durante rotação, a API aceita chaves anteriores sob `InvitationProtection:PreviousHashKeys:<id>` ao lado da chave ativa. O conjunto continua privado e isolado por ambiente. [Contratos](../spec.md#5-contratos-de-api)
