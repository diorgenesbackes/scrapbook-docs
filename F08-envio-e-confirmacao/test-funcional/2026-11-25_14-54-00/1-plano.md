# Plano de aceite local da F08-STORY-002

Validar um depoimento com uma foto em Docker, PostgreSQL e S3 versionado com SQS descartáveis. Usar dois eventos, aviso e arquivos sintéticos, relógio controlado e os candidatos da story identificados por repositório. Repetir a confirmação de texto com a mídia desativada. [Cartão](https://trello.com/c/EeNKYoWn) · [Especificação](../../spec.md)

| Critério | Cenário e evidência esperada |
| --- | --- |
| CA1 | Aceitar uma foto JPEG, PNG e HEIC/HEIF e legenda até 500 escalares. Exercitar manifesto inválido e os limites de tamanho e checksum. Conferir decodificação e hash dos bytes da versão fixada. |
| CA2 | Testar seleção, captura, envio por teclado, progresso de bytes, validação separada e recuperação da mesma tentativa após interrupção. Conferir validade e campos assinados da autorização e isolamento da tentativa. |
| CA3 | Admitir antes do fechamento de novos envios. Entregar evidência após o limite para uma versão criada às 13h exatas. Exercitar uma versão tardia e verificar ausência de confirmação até a decisão de reconciliação. |
| CA4 | Forçar falha entre salvar o original e confirmar no PostgreSQL, recuperar a cópia existente e conferir a unicidade do registro e dos fatos. Repetir evidência, enviar uma versão posterior e sanear somente temporários elegíveis, preservando o original e o recibo. |

A fixture combina PUT com bytes reais e eventos sintéticos com `VersionId` observado no armazenamento. Para confirmar o transporte real em homologação, verificar o envelope nativo da AWS, CORS, IAM, acesso negado à leitura pública, recuperação e atraso da limpeza no perfil exercitado. O resultado local mede o candidato e a fixture em loopback. [Dependências e estratégia de teste](../../spec.md#7-estratégia-de-testes)
