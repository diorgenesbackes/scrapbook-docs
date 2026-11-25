# Jornada de envio de texto e recuperação

Em 25/11/2026, os candidatos integrados da API `b0ede23`, UI `9c4a570` e testes `1623a1b` passaram nos 15 cenários de `submission_stack.py`. O Playwright percorreu três jornadas em Chromium, Firefox e WebKit desktop e em Android e iPhone emulados. [Plano de aceite](../test-funcional/2026-11-25_10-24-40/1-plano.md)

| Jornada | Evidência |
| --- | --- |
| Confirmar por teclado | Houve um único POST de contribuição após a ação explícita. O foco chegou ao recibo. O recibo passou no axe e omitiu o texto e a autoria. Novo depoimento abriu vazio. |
| Recuperar após perda de resposta | O POST recebeu `201` no proxy de teste. A janela foi encerrada e a resposta foi descartada. O navegador consultou a tentativa com a mesma chave e a mesma credencial e exibiu o recibo, desabilitando novo ingresso. |
| Renovar e descartar | A prova expirada retornou à identificação com a prévia preservada. Após renovar, a saída apresentou o diálogo com foco em Permanecer. Escape preservou a prévia. O descarte e a troca de evento abriram o editor vazio. |

Um cenário adicional de Android emulado mediu cinco aberturas com cache frio. Os testes usaram dados sintéticos. Dispositivos físicos e leitores de tela dependem do ensaio em homologação. [Medições](../test-performance/f08-story-001.md)

A regressão `testimonial_stack.py` acrescentou 20 cenários de composição, 15 de página pública e compartilhamento e uma medição 4G da revisão. Os 36 passaram. A stack com envio desabilitado registrou zero contribuições persistidas. No total, passaram 50 cenários funcionais de navegador e dois cenários de medição.
