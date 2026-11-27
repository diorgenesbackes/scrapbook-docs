# F10-STORY-001 — Mensageria e armazenamento

O template F10 configura uma assinatura do fato `record.media-confirmed.v1` num tópico SNS Standard existente. A fila de confirmação tem DLQ própria, criptografia gerenciada e política de entrega restrita ao tópico da conta. A configuração da F10 é independente da ativação do consumidor da F09. [Spec da F10](../spec.md#2-impacto-na-arquitetura)

A atualização da stack F08 adiciona o contexto KMS do prefixo de derivados. A política F10 permite à role da tarefa ler versões originais, verificar versões derivadas e criar variantes sob `submissions/derived/` com escrita condicional. Para testar a operação, a identidade de homologação deve confirmar a leitura da versão registrada, a ausência de acesso anônimo e o escopo de KMS e SQS no ambiente real. [Escrita condicional no S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-writes-enforce.html)

A suíte local de configuração passou com 17 testes, incluindo quatro novos para roteamento, redrive, acesso por versão, escrita condicional e KMS. O teste de reserva de release emitiu o aviso esperado ao criar os cinco remotos Git vazios de fixture. Os scripts de publicação passaram em `bash -n`.
