# Aceite integrado da F08-STORY-003

O candidato executado foi service `c8c7ed9`, UI `a46fc57`, tests `81d85ea`, docs `1c0db35`, Liquibase `a7cbec2` e infraestrutura `8a0f217`. O laboratório descartável usou PostgreSQL 18.6, Liquibase, S3 versionado e fila de eventos emulados por Moto e aviso sintético. O relógio inicial da aplicação partiu da hora do contêiner de serviço, cerca de 30 s atrás do host. As asserções do runner HTTP permanecem as mesmas. [Plano](1-plano.md) · [Especificação](../../spec.md)

A execução de 25/11/2026 terminou com código 0 nas três verificações, com Node 26.13.0 e Playwright 1.65.0. Os scripts temporários ficaram fora dos checkouts integrados.

| Verificação | Resultado |
| --- | --- |
| Runner HTTP do conjunto `python3 -B runtime/e2e-moments.py` | 1 suíte aprovada em 44,756 s, com o relógio inicial adaptado para a VM. [Relatório HTTP](http.txt) |
| Navegador, banco e bytes dos originais `python3 -B runtime/e2e-moment-browser.py` | 6 de 6 jornadas aprovadas em 35,5 s, com zero skips e 64,211 s incluindo a stack e verificações persistentes. [JUnit](moment-browser.xml) |
| Regressões `python3 -B runtime/e2e-legacy-regressions.py` | 2 suítes de integração aprovadas em 93,741 s, com 10 de 10 jornadas no navegador. [Relatório de regressões](regressoes.txt) |

| Critério | Resultado local |
| --- | --- |
| CA1 | HTTP admitiu conjuntos de uma e dez fotos, aceitou legenda com 500 escalares e recusou os sete manifestos inválidos planejados. O navegador selecionou, revisou e enviou duas fotos com legenda comum. A requisição trouxe o manifesto na ordem da revisão e com os checksums dos bytes selecionados. |
| CA2 | HTTP manteve a tentativa em recebimento, com apenas a última posição pendente, e recusou a autorização da foto reconhecida, de uma chave desconhecida e em outro evento. Ao interromper o segundo PUT no navegador, a tela indicou apenas **Foto 2**. **Tentar novamente** reaproveitou a chave da tentativa e a credencial, sem autorizar ou transferir a foto 1 de novo. |
| CA3 | HTTP confirmou quando a evidência do último arquivo marcou exatamente 13h de Brasília, mesmo processada depois. Uma evidência 1 ms posterior expirou o conjunto. Uma foto com conteúdo inválido rejeitou o conjunto, identificou a posição 2 e deixou zero confirmações e fatos de mídia. No navegador, o seletor recusou um arquivo sem imagem e o servidor rejeitou o JPEG truncado na posição 2 com `photo-invalid`. |
| CA4 | A confirmação retornou o recibo genérico com horário de Brasília, liberou as URLs de prévia e permitiu nova composição com a janela aberta. A repetição retornou o mesmo recibo. Quatro tentativas válidas no navegador tiveram um registro, uma operação idempotente, um fato de primeira confirmação e dois fatos de mídia por evento. Os dois eventos com rejeição ficaram sem confirmação, fatos ou referências a originais armazenados. A consulta das oito versões privadas no Moto confirmou seus bytes e ordem. O runner HTTP observou a exclusão das versões temporárias elegíveis nos conjuntos rejeitado e expirado. |
| Regressões | Dez jornadas das suítes existentes passaram com Chromium desktop a 320 px e Android emulado, seis de texto e quatro de depoimento com foto. Os fluxos HTTP também validaram prazo, idempotência, concorrência, recuperação, persistência e Outbox conforme a suíte de cada formato. |

Na primeira rodada de navegador, os quatro cenários de sucesso e retentativa passaram. O PNG de 12.001 × 1 foi rejeitado corretamente na posição 2 nos dois perfis, com `photo-invalid`. A asserção do ensaio esperava `photo-resource-budget` para essa amostra. A especificação exige uma causa estável e a posição, sem fixar classificação para essa dimensão. Para validar a causa de conteúdo com uma amostra explícita, o reteste usou um JPEG sintético sem o marcador de fim. O Chromium exibiu sua prévia e o servidor classificou o conteúdo como inválido.

A validação ficou no ambiente local, com arquivos sintéticos e eventos de recebimento introduzidos na fila de Moto após cada PUT observado. Homologação ainda verifica o aviso oficial, S3/SQS reais, IAM, KMS, CORS e o atraso operacional do saneamento na AWS. A carga e a segurança são registradas nas etapas próprias. [Estratégia de testes](../../spec.md#7-estratégia-de-testes)
