# Navegador e integração da identificação lembrada

A execução integrada de `python3 -B guest_identification_stack.py` em 24/11/2026 avaliou o frontend `68aeafd`, os testes `ecb2877` e o serviço `76fd9e0`. O Docker usou PostgreSQL 18.6, dois eventos sintéticos abertos, relógio controlado e consultas reais à página pública da F05.

Playwright 1.65.0 aprovou 35 casos nos cinco perfis Chromium, Firefox, WebKit, Android emulado e iPhone emulado. Houve 20 verificações da nova story e 15 regressões da primeira identificação, sem falhas nem casos ignorados. Foram exercitados retorno ao mesmo evento, alternância entre prévias, edição, troca de evento, remoção do registro, acesso ao armazenamento bloqueado e registros incompatíveis. A recuperação conservou o rascunho da página em ambos os cenários.

Os testes de componente verificaram a entrega dos campos confirmados ao consumidor da F06. As entradas reais da F07, o envio da F08, aparelhos físicos e leitores de tela ficaram fora da execução. [Contrato](../dev-front/f06-story-002.md) · [Aceite local](../test-funcional/2026-11-24_11-53-47/2-execucao.md)
