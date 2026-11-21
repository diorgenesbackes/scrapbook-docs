# F03-STORY-002 · Política de abertura e auditoria

O `PUT /api/v1/events/{eventId}/opening-policy` recebe modo, data civil ou nula e `expectedVersion`. A operação exige identidade, origem autorizada e vínculo. A Service lê a linha com bloqueio de atualização e só depois consulta o relógio. Compara a versão, reavalia a trava em Brasília e decide a transição.

Antes de D+1 às 12h, a política permanece flexível. Na borda, o modo futuro torna-se efetivamente travado. Se estiver no modo imediato, uma última troca para futuro grava a trava e o marco da escolha na mesma escrita. Às 13h, encerra-se apenas a tolerância dos envios iniciados. Um envio sem mudança mantém a versão e a trilha, inclusive após o fechamento.

Uma mudança incrementa a versão do evento e a versão da política. A auditoria registra a conta, o evento, o instante UTC, a correlação e os estados anteriores e novos em uma única transação com a mudança. A retenção é de 180 dias. A identidade DML insere o registro sem precisar de permissão de leitura da auditoria. O catálogo da F01 e a geração da criação de eventos fornecem as colunas usadas. [Transições e contrato](../spec.md#transições-da-abertura)

A criação agora aceita imediato, com data nula, e mantém a sugestão de um ano para futuro. O fingerprint de solicitações futuras conserva a representação anterior para recuperar tentativas existentes. [Repetição da criação](../spec.md#evolução-para-repetição-segura)
