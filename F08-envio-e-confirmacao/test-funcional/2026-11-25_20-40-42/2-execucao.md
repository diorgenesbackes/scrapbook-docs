# Execução local da F08-STORY-003

A verificação exerceu os serviços de aplicação e repositórios reais em PostgreSQL 18.6 descartável no loopback, com três changesets Liquibase aplicados. O ensaio usou relógio controlado, storage versionado simulado, decoder simulado e dados sintéticos. O restore bloqueado pelo lockfile, build e testes .NET passaram, com 199 testes. O lint, build e os 135 testes do frontend passaram.

| Critério | Resultado observado | Evidência pendente |
| --- | --- | --- |
| CA1 | Unitários exercitaram quantidade, UUIDs distintos, posições consecutivas, ordem, tamanho e legenda. O banco persistiu as dez posições e a legenda comum na ordem prevista. | Jornada do navegador. |
| CA2 | Uma prova parcial retirou sua posição do status. O serviço recusou autorizar a posição já reconhecida e uma chave ausente. Os testes de UI observaram bytes, preservaram a credencial e retomaram somente a posição pendente. | PUT real no navegador e isolamentos HTTP com armazenamento versionado. |
| CA3 | Provas simultâneas selaram o conjunto. O instante de 13h exatas foi aceito. A consulta passou para reconciliação no corte. A ausência e o atraso impediram a confirmação. Falhas de conteúdo e orçamento foram associadas a posições distintas. | Recepção pela fila e pelo emulador de S3. |
| CA4 | O banco persistiu dez referências a originais e dez fatos de mídia. A repetição recebeu o mesmo recibo. Uma falha da Outbox reverteu a confirmação inteira. A repetição após fim do lease gravou os fatos e reaproveitou as cópias. O ensaio solicitou a exclusão de 16 versões temporárias e preservou as referências a originais. | Exclusão física no armazenamento versionado e regressões ponta a ponta. |

O daemon Docker ficou sem resposta à sondagem local. O teste manual `python3 -B moment_photo_submission_stack.py` foi preparado e teve sua entrada verificada com `--help`. A execução completa da fixture com HTTP, S3 e PostgreSQL, os cenários Playwright, carga e segurança da versão integrada permanecem pendentes. [Plano](1-plano.md) · [Especificação](../../spec.md)
