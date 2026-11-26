# F09-STORY-001 — Aplicação da trava estrutural

A F08 já grava `event.first-record-confirmed.v1` no commit de confirmação de texto ou mídia. O publicador da F09 reivindica esse tipo na Outbox, envia o envelope ao SNS e registra publicação após aceite do transporte e com a mesma lease. Reentregas preservam o `messageId` lógico. [Cartão](https://trello.com/c/cWad9rrV) · [Spec](../spec.md#5-contratos-de-api-e-mensagens)

O consumidor confere o envelope com o fato persistido e o instante de confirmação no PostgreSQL. Na transação de efeito, bloqueia a deduplicação, o evento e o convite nessa ordem. Aplica o primeiro registro e o instante de trava, incrementa a versão do evento e invalida o convite pendente, limpando seu ciphertext e a chave de cifragem. Registra auditoria sistêmica correlacionada e conclui a deduplicação antes de confirmar a entrega no SQS. O processamento preserva os vínculos e a política de abertura.

O contrato reutiliza os campos e grants existentes de eventos, convites, Outbox, deduplicação e auditoria. Ao ativar o fluxo, o serviço aceita fila e tópico configurados e usa a configuração local do emulador em desenvolvimento. A story seguinte cobre a recuperação operacional e os resultados estruturados para F13. [Dados](../spec.md#6-modelo-de-dados) · [Mensageria](../dev-infra/f09-story-001.md) · [Recuperação](https://trello.com/c/fzQlYzmM)
