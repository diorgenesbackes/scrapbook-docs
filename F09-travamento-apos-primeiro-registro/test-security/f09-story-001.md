# F09-STORY-001 — Verificação defensiva

A análise independente examinou o candidato inicial `4f3f024` e repetiu os controles pertinentes com a API corrigida em `1ee9d816`. A interface `57f4e63`, as suítes `7138d3d` e a infraestrutura `56759f3` permaneceram fixas. O ensaio dinâmico usou identidades e eventos sintéticos em uma stack descartável. [Critérios da F09](../spec.md#7-estratégia-de-testes)

| Controle | Resultado |
| --- | --- |
| Autorização e isolamento | Titular e corresponsável mantiveram a leitura e as ações autorizadas. Sem sessão, o detalhe respondeu `401`. Conta de fora do vínculo recebeu `404`. Detalhes e convites incompatíveis com a trava receberam `409`. |
| Contratos e privacidade | Respostas privadas, públicas e Problem Details passaram nas verificações de minimização. A resposta incluiu correlação e aplicou `no-store` onde exigido. O navegador exibiu mensagens neutras e removeu o QR. A auditoria representou a transição da trava e o consumidor registrou códigos técnicos. |
| Proveniência | O consumidor conferiu a identidade estável, o registro e o instante contra dados do próprio evento na Outbox e no banco. O ACK ocorre após o commit que grava o efeito, a auditoria e a deduplicação. |
| IAM e configuração | O template limitou a entrega ao SNS da conta e do tópico esperados, restringiu a role da tarefa ao tópico e à própria fila e habilitou criptografia de fila e controles de TLS. A configuração publicada exige URL regional HTTPS de SQS na conta do tópico. |

A inspeção encontrou correlação vazia em fatos elegíveis da F08. A API passou a gerar um UUID nessa condição na entrada e os dois lados do transporte usam o `messageId` como correlação estável para fatos históricos com valor vazio. A revisão de código e quatro regressões unitárias confirmaram a correção. [PR da API](https://github.com/diorgenesbackes/scrapbook-service/pull/13)

O primeiro runner HTTP comparou o cabeçalho de cache inteiro e interrompeu o subteste ao receber `no-cache,no-store`. A asserção passou a conferir as diretivas de cache. O reteste com o mesmo candidato integrado passou em 29,303 s. [Evidência E2E](../test-e2e/f09-story-001.md)

As permissões efetivas de IAM, TLS, Cognito e o transporte nativo continuam sujeitos ao aceite em homologação. A recuperação de falhas definitivas e o redrive operacional pertencem à F09-STORY-002. [Template](../../../scrapbook-infrastructure/config/messaging/event-structural-lock.template.json)
