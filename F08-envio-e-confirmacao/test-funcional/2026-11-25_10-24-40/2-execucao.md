# Aceite local da F08-STORY-001

Em 25/11/2026, o ensaio integrado usou API `b0ede23`, UI `9c4a570`, testes `1623a1b` e PostgreSQL descartável com as migrations vigentes. [Plano](1-plano.md) · [Cartão](https://trello.com/c/r9abSKt6)

| Critério | Evidência | Resultado |
| --- | --- | --- |
| CA1 | O HTTP serviu o aviso vigente e recusou provas de outro evento, nome alterado e versão anterior. A prova vencida foi recusada e o Playwright verificou renovação preservando o texto. | Aprovado |
| CA2 | O HTTP rejeitou o instante anterior à abertura, admitiu na abertura, rejeitou nova tentativa no fechamento e verificou autoria e instante civil no PostgreSQL. Os unitários admitiram recebimento exatamente às 13h e recusaram o primeiro tick posterior. Os limites de 5.000 e 5.001 escalares foram exercitados. | Aprovado |
| CA3 | Oito pedidos da mesma tentativa produziram um `201`, sete `200` e uma única gravação. Quatro tentativas distintas confirmaram quatro registros com um fato inicial. Uma falha da Outbox reverteu todas as escritas e permitiu confirmar ao reapresentar a tentativa. | Aprovado |
| CA4 | O status e o replay autorizado devolveram o mesmo recibo após o fechamento. A chamada com conteúdo divergente retornou `409`. O navegador recuperou o recibo sem repetir o POST após a perda da resposta. | Aprovado |
| CA5 | O envio explícito anunciou a validação e colocou o foco no recibo. Novo ingresso ficou disponível com a janela aberta. Descarte e navegação protegeram o rascunho e os logs omitiram os marcadores privados. | Aprovado |

Passaram 140 unitários da API e 122 da UI. A matriz de navegador, os resultados de carga e o recorte de segurança estão registrados nas respectivas etapas. [E2E](../../test-e2e/f08-story-001.md) · [Carga](../../test-performance/f08-story-001.md) · [Segurança](../../test-security/f08-story-001.md)

A regressão de composição, página pública e compartilhamento também passou. Com o envio desabilitado, compor, revisar e descartar continuam sem persistir contribuições. [Regressão de navegador](../../test-e2e/f08-story-001.md)
