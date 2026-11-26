# Verificação defensiva da F08-STORY-002 integrada

A revisão independente verificou as árvores integradas e seus SHAs no relatório de entrega. Passaram 179 unitários do serviço, com zero falhas e ignorados, e nove verificações da infraestrutura. O restore usou pacotes locais fixados pelo lockfile. [Entrega](../delivery/f08-story-002.md)

A integração em loopback verificou que segredo divergente, evento diferente e arquivo fora da tentativa recebem HTTP 404. O PUT usa origem, tipo, tamanho e SHA-256 vinculados. O recibo tem somente estado e horário. A versão posterior do temporário não substituiu o original confirmado e a limpeza removeu a versão temporária elegível. Os logs não continham os marcadores privados da fixture. O teste usou Moto com prova sintética e o identificador real retornado pelo PUT. [Aceite integrado](../test-funcional/2026-11-25_22-32-28/2-execucao.md)

A inspeção encontrou um impedimento de recuperação da DLQ em `8a0f217`. A fila de recibos nega publicação sem o ARN e a conta do bucket. O redrive nativo exige `sqs:SendMessage` no destino, em chamada mediada pelo SQS em nome da identidade operacional. A promoção considera a correção dessa política antes da ativação de fotos. [Redrive no SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-dead-letter-queue-redrive.html)

IAM, KMS, entrega nativa, acesso público efetivo, isolamento do decoder e redrive exigem verificação no ambiente de homologação. [Validação local anterior](f08-story-002.md)

## Correção e reteste

A correção permite o retorno mediado pelo SQS em nome de uma role operacional dedicada, com a mesma conta e escopo de duas filas. A configuração é opcional. Sem a role, a fila mantém a origem S3 prevista. A política rejeita envio direto da role operacional e mantém a role da tarefa separada, inclusive diante de variações de capitalização. O encaminhamento automático à DLQ usa as mesmas propriedades das filas.

A revisão independente executou 136 asserções offline com a configuração habilitada e desabilitada, identidades e fontes divergentes, precedência das negativas e os limites de origem e destino. Passaram também os nove testes da infraestrutura e `cfn-lint` 1.61.0. A matriz local verifica as condições declaradas. Em homologação, comprovar a entrega nativa, a transferência à DLQ após as tentativas, o retorno da mesma prova pelo operador e a recusa de publicação direta. [PR da correção](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/2)
