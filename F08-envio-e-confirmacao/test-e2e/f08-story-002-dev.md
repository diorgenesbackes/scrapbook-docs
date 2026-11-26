# Jornada integrada de depoimento com foto

Em 25/11/2026, a F08-STORY-002 integrada em `dev` passou em quatro casos Playwright com Chromium `159.0.8154.6`, desktop a 320 px e Android emulado, Playwright `1.65.0`, Node `26.13.0` e npm `11.20.0`. O navegador executou 32,5 s, com zero falhas, skips ou retries. [JUnit](f08-story-002-dev-junit.xml) · [Aceite funcional](../test-funcional/2026-11-25_22-32-28/2-execucao.md)

| Critério | Asserções observadas nos dois projetos |
| --- | --- |
| CA1 e CA2 | Selecionar o PNG, revisar legenda de 500 escalares, enviar pelo teclado e medir bytes transferidos. A validação ficou visível antes do recibo. O recibo recebeu foco, mostrou hora de Brasília e passou no axe. O novo depoimento abriu com prévia e legenda limpas, sem rolagem horizontal. |
| CA2 e CA3 | Capturar JPEG pela câmera emulada. Após a interrupção do primeiro PUT, renovar a autorização sem criar outra tentativa. Confirmar a foto com evidência de recepção às 13h após fechar a janela. O recibo omitiu nome, legenda, foto e ações de leitura ou exclusão. |

A execução inicial de `python3 -B photo_submission_stack.py --browser` terminou com código 1 antes do navegador. A fixture usava a hora do host e a assinatura usava o relógio da VM, cerca de 30 s atrasado. A primeira URL declarou 629 s. O reteste usou um adaptador temporário para ancorar a hora inicial da fixture no contêiner de serviço e preservou todas as asserções. As autorizações verificadas ficaram entre 567 e 599 s. A integração e a limpeza dos recursos próprios terminaram com código 0 em 67,250 s. [Execução e revisões](../test-funcional/2026-11-25_22-32-28/2-execucao.md)

O alvo foi Docker dedicado em loopback com PostgreSQL, Moto e mensagens SQS sintéticas. O `VersionId` das mensagens veio da resposta real do PUT local. O ensaio de foto cobriu câmera emulada e deixou os dispositivos físicos para validação manual. A execução local também deixou para homologação a notificação nativa do S3, IAM, KMS e CORS do ambiente. [Spec](../spec.md#7-estratégia-de-testes)
