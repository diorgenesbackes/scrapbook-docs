# Aceite local da F08-STORY-002 integrada em dev

O reteste de 25/11/2026 passou na integração HTTP, PostgreSQL e Moto/SQS e nas quatro jornadas Playwright. Validaram-se T11 e T12 no ambiente local dedicado. O navegador concluiu em 32,5 s e a suíte completa em 67,250 s, com código 0. [Plano](1-plano.md) · [Jornadas e evidência JUnit](../../test-e2e/f08-story-002-dev.md)

| Repositório | SHA de `dev` validado |
| --- | --- |
| Serviço | `43d2af8fd239972294947810b530551938c01049` |
| UI | `f0aa343a134934181a364a4e74b9c331e3de618e` |
| Testes | `7225c5a7be1b69b0198b3a6cb260271210dfbcb4` |
| Docs | `3ad9c079001c301679ee315c7687eae33653f1c2` |
| Liquibase | `a7cbec27d2255cfa43fa5a77ae72f0e3f1fc04f5` |
| Infraestrutura | `8a0f2172c42063814e45b046a268a1581251f23a` |

| Critério | Resultado integrado |
| --- | --- |
| CA1 · F07 → F08 | PNG, JPEG e HEIC confirmaram após decodificação. Manifesto sem arquivos, tamanho de 20.000.001 bytes, legenda de 501 escalares e bytes inválidos foram recusados. Legenda de 500 escalares passou no navegador. O SHA-256 e os bytes do original conferiram com o manifesto. O candidato anterior exercitou 20.000.000 bytes, o MIME `image/heif` e checksum incorreto em testes unitários. A execução integrada usou fotos menores e a fixture `image/heic`. [Aceite anterior](../2026-11-25_14-54-00/2-execucao.md) |
| CA2 · S25 e S26 | PUT da chave planejada criou versões distintas com comprimento, tipo e SHA-256 assinados. Credencial incorreta, evento diferente e chave de arquivo desconhecida foram recusados. A mesma tentativa retomou após interrupção. O progresso de bytes, o estado de validação e o recibo apareceram em momentos diferentes. [E2E](../../test-e2e/f08-story-002-dev.md) |
| CA3 · F05 → F08 | Nova admissão às 12h e nova autorização depois das 13h foram recusadas. Uma evidência tardia confirmou a versão recebida às 13h exatas. A versão de 13h mais 1 ms ficou fora da confirmação e expirou após reconciliação. A interface já esperava a evidência antes de apresentar confirmação. [Spec](../../spec.md#3-decisões-técnicas) |
| CA4 · F08 → F09, F10 e F11 | A falha injetada na transação de Outbox deixou a cópia sem confirmação. A recuperação reutilizou a versão copiada. Duplicata e PUT posterior preservaram o registro, os fatos existentes e a versão fixada. O saneamento manteve as versões recentes, removeu as elegíveis ao avançar o relógio de reconciliação e preservou o original privado. [Spec](../../spec.md#primeira-confirmação-e-obrigações) |

O comando inicial `DOCKER_CONTEXT=colima-scrapbook-f08-validation python3 -B photo_submission_stack.py --browser` parou antes do PUT com código 1 em 35,933 s. O `X-Amz-Expires` observado foi 629 s, acima da asserção de 600 s. O reteste cotejou a hora do host (`01:35:06.611–01:35:06.672Z`) com a do contêiner (`01:34:36Z`) e ancorou o relógio inicial da fixture no contêiner. O adaptador ficou fora dos repositórios. Todas as asserções originais foram mantidas. O reteste observou autorizações de 567 a 599 s e 947 ms entre enviar a evidência de uma foto PNG de 922.428 bytes e observar o recibo. Trata-se de uma amostra funcional do laboratório. O runner verificou ausência dos marcadores privados da fixture nos logs da API e da UI.

O ensaio cobriu PUT, versões e cópias no Moto, com mensagens SQS sintéticas associadas a versões reais da fixture. O prazo operacional da limpeza após 24 h, a origem da notificação e a precisão do evento nativo, os controles IAM/KMS e CORS publicado continuam para homologação. Carga e segurança defensiva pertencem aos ensaios próprios. A regressão integral de texto permanece separada desta execução. [Spec](../../spec.md#7-estratégia-de-testes)
