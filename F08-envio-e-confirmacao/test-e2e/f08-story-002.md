# Jornada de depoimento com foto

A execução independente de 25/11/2026 usou serviço `20097b0`, UI `c8005e7`, testes `068add6`, infraestrutura `8d9a753` e Liquibase `ef51987`. O comando `python3 -B photo_submission_stack.py --browser` passou nos quatro cenários Playwright em Chromium desktop e Android emulado. [Plano de aceite](../test-funcional/2026-11-25_14-54-00/1-plano.md)

| Cenário | Evidência |
| --- | --- |
| Selecionar e enviar pelo teclado | O manifesto continha uma foto PNG e legenda de exatamente 500 escalares. A interface exibiu progresso real, anunciou a validação e focou o recibo. Novo depoimento abriu com seletor e legenda limpos. O axe passou no recibo. |
| Capturar, interromper e repetir | A captura forneceu uma foto JPEG. O PUT interrompido foi retomado com a mesma tentativa e a confirmação foi recuperada após o fechamento. A evidência de criação ficou na fronteira de 13h em Brasília. |

O primeiro ensaio detectou rolagem horizontal ao reiniciar o seletor em 320 px. A correção `386754d` igualou a largura rolável à viewport e o cenário passou no reteste. A evidência temporal local usa a versão realmente devolvida pelo Moto e mensagem SQS sintética. [Review](../code-review/f08-story-002.md) · [Resultados funcionais](../test-funcional/2026-11-25_14-54-00/2-execucao.md)

A regressão `python3 -B submission_stack.py` passou em 15 casos de navegador e no ensaio 4G Android. As cinco aberturas chegando ao editor de texto ficaram em até 2.044 ms e os cinco recibos em até 231 ms. No k6, os 200 envios de texto passaram, com p95 de 187,41 ms e zero falhas HTTP. [Fluxo de texto](../test-e2e/f08-story-001.md)
