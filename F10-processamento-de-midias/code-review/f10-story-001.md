# Revisão técnica da F10-STORY-001

A revisão independente examinou o serviço `9563ad4`, Liquibase `1299de1`, infraestrutura `a8c40bd` e testes `f9cac02`. A comparação partiu das respectivas branches `dev` sincronizadas. [Cartão da story](https://trello.com/c/3tAcFFKS) · [Spec da F10](../spec.md)

## Correções e verificação

- Depois de timeout ou falha de transporte do SNS, o publicador verifica a admissão já confirmada na mesma transação em que decide o estado da Outbox. Um resultado incerto fica sinalizado para reconciliação. A rejeição inequívoca pode produzir o fato de falha para a F13.
- A inicialização recusa a fila de mídia quando coincide com a fila de recebimento da F08 ou a fila ativa da F09. As assinaturas podem compartilhar o tópico SNS.
- A fixture de retomada encerra apenas as sessões PostgreSQL da aplicação que aguardam o lock do ensaio depois da queda das duas réplicas. A fixture confere o vínculo do fato de falha pelo agregado e pelos identificadores de mensagem, registro e arquivo.

A revisão final deixou o candidato sem achados técnicos bloqueantes. Na verificação das permissões de homologação, avaliar a restrição efetiva de rede e de sistema de arquivos do subprocesso. O laboratório usa a política de formatos, delegados e recursos herdada da F08. [Validação de segurança da F08](../../F08-envio-e-confirmacao/test-security/f08-story-002.md)

## Checks locais

| Check | Resultado |
| --- | --- |
| .NET 10.0.401, restore locked, build e xUnit | Build com zero warnings e 447 testes aprovados, sem casos ignorados. |
| `dotnet format --verify-no-changes --no-restore --severity info` | Código 2. A solução tem 24 diagnósticos de espaço em branco preexistentes. A revisão corrigiu as três ocorrências introduzidas. O relatório final registra sugestões informativas, inclusive para uso de desconstrução em testes. |
| Frontend `57f4e63`, Oxlint, Vite e Vitest | Lint e build aprovados, 188 testes aprovados. O Vitest informou tempo de inicialização do jsdom. |
| Liquibase e infraestrutura | 31 testes Liquibase e 17 testes Python aprovados. O teste de reserva de releases emitiu avisos esperados sobre remotos vazios da fixture. |
| `git diff --check` | Aprovado nos quatro repositórios revisados. |

O coletor da suíte registrou 55,2% nas linhas executáveis alteradas como parte da story e 83,6% nas linhas da correção técnica. Asserções de PostgreSQL, entrega SQS e execução do subprocesso estão na suíte integrada.
