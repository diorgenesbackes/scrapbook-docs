# Revisão técnica da emissão de convite

| Área | Evidência |
| --- | --- |
| API e regras | Identidade, vínculo, vaga e trava precedem a entrega do segredo. O serviço lê o relógio após obter o lock do evento. |
| Persistência | O serviço coordena emissão e auditoria. Operações do repositório persistem apenas os valores recebidos. O ensaio força falha de auditoria e confirma rollback da substituição. |
| Criptografia | Chaves distintas e envelope com nonce aleatório. O teste comprova reexibição e recusa de envelope ligado a outro evento, outra operação ou outra chave de ambiente. |
| Interface | Foi conferido o foco ao confirmar ou cancelar, a recuperação com a mesma chave e a apresentação da URL canônica. |
| Hospedagem | A entrada mantém `no-referrer` e `noindex, nofollow`. A interface reutiliza o cliente HTTP, o gerador de QR e o hook de cópia. |

Durante a revisão, os estados anterior e posterior da troca de convite passaram a ser registrados separadamente. O teste de validação de idempotência passou a enviar uma chave inválida em ASCII. O cabeçalho com texto acentuado era recusado pelo servidor HTTP antes da validação de UUID. [Contrato](../spec.md#5-contratos-de-api)

A emissão normaliza a precisão do horário para microssegundos antes de devolver o vencimento. O unitário usa um relógio com precisão maior e verifica que a resposta inicial conserva o mesmo prazo depois de consultar ou repetir a emissão.

Os checks passaram em 116 unitários do serviço e 47 da interface. O build .NET ficou com zero avisos e erros. `npm ci --offline` emitiu o aviso referente ao script opcional de `fsevents`. Lint, build e testes passaram. O pacote de publicação mantém a proteção da entrada do convite, e os 61 cenários simulados de publicação e recuperação continuaram aprovados.
