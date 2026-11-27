# F10-STORY-002 — Entrega para homologação

A segunda story prepara áudios confirmados pela F08 e publica uma referência M4A/AAC privada no contrato interno. A verificação por versão e a reconciliação preservam o recibo e o original. [Cartão](https://trello.com/c/I4MI0oAu) · [Spec](../spec.md)

| Componente | Integração em dev | SHA correspondente |
| --- | --- | --- |
| Serviço | [PR #19](https://github.com/diorgenesbackes/scrapbook-service/pull/19) | `d7a6a4ba34cbf83ed6976ea2c55713c8f3fdef1e` |
| Infraestrutura | [PR #15](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/15) e [correção IAM #16](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/16) | `b8c825736a5e065d78bd122c384ff373107d6182` |
| Testes | [PR #18](https://github.com/diorgenesbackes/scrapbook-tests/pull/18) | `6338f1efec19c7ccbd5fe12f09e8ba7c224f9554` |
| Documentação inicial | [PR #26](https://github.com/diorgenesbackes/scrapbook-docs/pull/26) | `d3c9700ddef5cf1ab4b02e287b24bc99a69f5510` |

A entrega depende da F08 de áudio e do schema e da admissão da F10-STORY-001. A série não introduz changeset nem alterações de UI. A promoção seleciona somente os commits da story e da correção IAM. Os candidatos de serviço, infraestrutura e testes têm o mesmo conteúdo das versões em `dev` validadas. Os SHAs dos merges e o resultado da publicação ficam registrados no cartão. [Serviço #20](https://github.com/diorgenesbackes/scrapbook-service/pull/20) · [Infraestrutura #17](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/17) · [Testes #19](https://github.com/diorgenesbackes/scrapbook-tests/pull/19) · [Cartão](https://trello.com/c/I4MI0oAu)

## Evidências locais

- Build sem warnings, 577 testes unitários do serviço, 188 da UI, 21 verificações de infraestrutura e revisão independente. [Revisão técnica](../code-review/f10-story-002.md)
- Cinco formatos F08, três cenários de espera, recuperação e falha e quatro jornadas Playwright. A rodada usou PostgreSQL e uma fixture do contrato MediaConvert. [E2E](../test-e2e/f10-story-002-dev.md) · [Aceite funcional](../test-funcional/2026-11-27_14-22-37/2-execucao.md)
- Isolamento de recibos, verificação das versões privadas, recusa a leituras anônimas e reteste da política de confiança da fila. [Segurança defensiva](../test-security/f10-story-002-dev.md)

- Os 40 áudios da rajada local ficaram prontos, incluindo oito WebM normalizados. O p95 da confirmação até a observação do commit foi de 1,75 s. A fixture serviu uma referência M4A e o relatório detalha a defasagem de relógio observada. [Carga local](../test-performance/f10-story-002/f10-story-002-dev.md)

## Aceite em homologação

A sessão retornou `NoCredentials` antes dos ensaios em AWS. Manter `MediaAudioProcessing__Enabled` desabilitada até conferir a stack no ambiente, a assunção da role pela fila, o IAM efetivo, a criptografia, a escrita condicional e a entrega EventBridge/SQS, incluindo as DLQs. Disponibilizar pelo menos 1 GiB de espaço temporário gravável por réplica. [Infraestrutura e configuração](../dev-infra/f10-story-002.md)

Com amostras sintéticas de MP3, M4A, WAV, Ogg e WebM, verificar a leitura da entrada preparada pelo MediaConvert, coletar os tempos de espera e conversão, conferir a faixa, duração e metadados da saída M4A real e verificar a privacidade por versão. Em caso de pausa durante o aceite, desabilitar a flag e preservar as filas, tentativas, versões e originais para reconciliação. [Cenários da F10](../spec.md#7-estratégia-de-testes)
