# Plano de aceite integrado da F08-STORY-002

Validar a contribuição de depoimento com uma foto no resultado integrado em `dev`. Usar uma VM Colima dedicada, PostgreSQL com migrations do Liquibase, S3 versionado e SQS emulados pelo Moto, arquivo de relógio controlado e dois eventos com aviso sintético. Executar HTTP, persistência e as jornadas de seleção e câmera no Playwright. [Cartão](https://trello.com/c/EeNKYoWn) · [Spec](../../spec.md) · [PRD](../../../PRD.md#f08-envio-e-confirmação-1)

| Critério | Cenários e provas |
| --- | --- |
| CA1 · F07 → F08 | Decodificar e confirmar PNG, JPEG e HEIC. Recusar manifesto sem foto, tamanho de 20.000.001 bytes, legenda de 501 escalares e bytes sem imagem. Exercitar legenda de 500 escalares no navegador. Comparar os bytes da versão original com o SHA-256 do manifesto. |
| CA2 · S25 e S26 | Conferir seleção por teclado e câmera emulada, progresso real seguido de validação, interrupção e retomada com as mesmas credenciais. Verificar operação PUT, cabeçalhos assinados, limitação a 600 s e escopo do evento e arquivo. |
| CA3 · F05 → F08 | Recusar novas tentativas às 12h e novas autorizações após 13h. Confirmar a versão cuja notificação tardia informa o instante de criação exatamente às 13h. Excluir da confirmação versão criada 1 ms depois e observar expiração após reconciliação. |
| CA4 · F08 → F09, F10 e F11 | Observar recibo genérico após commit. Conferir bytes do original e unicidade do registro e da Outbox após duplicata e PUT posterior. Injetar falha de commit após a cópia e confirmar o reaproveitamento da versão. Aguardar saneamento elegível e verificar que a cópia confirmada permanece. |

A fixture publica mensagens SQS sintéticas com o `VersionId` devolvido pelo PUT real no armazenamento local. O aceite em homologação deve medir a origem e o horário do evento S3 nativo, IAM, KMS, CORS publicado, recuperação e atraso efetivo da limpeza. [Aceite do S3](../../spec.md#7-estratégia-de-testes)
