# F08-STORY-005 — Contrato de envio de áudio

O depoimento admite um arquivo MP3, M4A, WAV, Ogg ou WebM, com até 50.000.000 bytes e duração positiva até 300 s. A legenda pode conter até 500 escalares Unicode. A validação verifica o contêiner, a faixa de áudio, a decodificação e o SHA-256 integral da versão recebida. [Cartão](https://trello.com/c/KrtTW39U) · [Spec](../spec.md)

O áudio usa PUT direto no armazenamento privado e o mesmo contrato de tentativa das demais mídias. A admissão gera as intenções de temporário e original. A confirmação depende da prova de um PUT de versão exata até 13h de Brasília inclusive. Após a validação, o serviço preserva a versão privada e confirma arquivo, registro, sessão, operação idempotente e eventos Outbox na mesma transação. O recibo devolve o estado e o instante da confirmação.

O primeiro fato de confirmação do evento mantém o contrato com F09. O fato de mídia resolve o arquivo e o original privado para F10. A F11 pode consumir os registros confirmados e seus arquivos. F13 terá como origem durável a sessão de upload, com um resultado terminal, código, instante e correlação por tentativa. O código do resultado distingue conteúdo inválido ou orçamento esgotado, prova tardia e ausência de recepção comprovada. Repetições da transferência mantêm a mesma sessão.

A ativação usa `PhotoSubmissions__AudioEnabled=true` junto da configuração de ingestão existente. O schema de arquivos já aceita `kind = 'audio'` e possui colunas de contêiner, codec, duração e checksum. O áudio reutiliza o worker, a fila de recepção do PUT e a limpeza de versões temporárias. [Modelo](../spec.md#6-modelo-de-dados) · [Candidato anterior](../delivery/f08-story-004.md)
