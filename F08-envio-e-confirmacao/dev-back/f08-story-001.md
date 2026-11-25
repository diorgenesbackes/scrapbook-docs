# F08-STORY-001 — Admissão e confirmação de texto

A API pública serve o aviso vigente e emite uma prova de concordância válida por 30 minutos, vinculada ao evento, ao nome normalizado e à versão do aviso. A admissão usa o tempo registrado pelo servidor antes da leitura do corpo, a chegada integral e os limites de Brasília da F05. O texto preserva seus espaços, normaliza quebras de linha e aceita até 5.000 escalares Unicode. O limite HTTP é 64 KiB. [Contratos e erros](../spec.md#5-contratos-da-api)

O cliente fornece um UUID de tentativa e um segredo independente de 32 bytes. A API calcula HMACs por propósito e versão de chave. O status retorna estado e instante UTC por 30 dias. O replay autorizado devolve a confirmação anterior e detecta divergência de conteúdo. A unicidade da tentativa persiste após a expiração. [Modelo e rotação](../spec.md#6-modelo-de-dados)

O serviço de aplicação decide o primeiro fato sob lock da linha do evento e grava registro, sessão textual terminal, operação idempotente e o primeiro `event.first-record-confirmed.v1` em uma transação. O payload do fato contém IDs internos e o instante UTC. A API usa as tabelas e permissões já instaladas pelo Liquibase. [Modelagem da F08](../spec.md#6-modelo-de-dados)

A ativação requer `Submissions__Enabled=true`, `Submissions__KeyId`, `Submissions__Key` de 32 bytes em hexadecimal e um aviso vigente aprovado no ambiente. O runner fornece aviso e chave sintéticos efêmeros. `Submissions__PreviousKeys__<id>` permite manter chaves válidas para verificação durante rotação. A configuração padrão mantém o envio desabilitado. [Plano da F08](../plan.md)
