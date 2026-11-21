# F03-STORY-001 · API de criação e consulta

`POST /api/v1/events` cria o evento e o vínculo inicial no slot 1 com uma única gravação transacional. A Service calcula os marcos em `America/Sao_Paulo`, valida a data de abertura e gera o código público aleatório. A tentativa é vinculada à conta por um UUID v4. A gravação retém os hashes da chave e da entrada canônica. O replay com entrada equivalente devolve a representação atual se o vínculo ainda autoriza o acesso.

`GET /api/v1/events` lista os vínculos por ID decrescente, com página padrão de 20 e intervalo de 1 a 50. O cursor protegido identifica a conta e a posição. `GET /api/v1/events/{eventId}` usa o mesmo vínculo para limitar o detalhe. A ausência de vínculo e a ausência do recurso devolvem o mesmo `404`. Respostas protegidas usam `no-store`.

O laboratório usa identidades sintéticas de Development em loopback. Postagens exigem origem explícita, JSON estrito e até 16 KiB. A conexão e os comandos PostgreSQL usam o orçamento do ambiente. As operações propagam o cancelamento do cliente. A prontidão verifica as duas colunas da nova geração antes de admitir tráfego. [Spec](../spec.md#5-contratos-de-api) · [Banco](../dev-db/f03-story-001.md)
