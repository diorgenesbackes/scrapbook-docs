# F08-STORY-001 — Revisão, envio e recibo

A identificação de texto busca o aviso vigente e confirma a concordância com o servidor. A revisão habilita **Enviar** após receber a prova. Quando é preciso renovar o aviso, o convidado retorna à identificação com o texto revisado preservado. O fluxo local de identificação sintética continua disponível para os cenários anteriores de composição. [Estados da experiência](../spec.md#fluxo-da-interface)

O envio mostra o estado de andamento e consulta a mesma tentativa após perda de resposta. Uma retomada explícita consulta o status antes de reapresentar o mesmo pedido. O texto, a prova e as credenciais de envio ficam em memória. O rascunho mantém a proteção de saída enquanto há envio a recuperar.

O recibo libera o rascunho e reconsulta a janela pública para oferecer **Deixar outro depoimento** e **Registrar um momento**. O horário de confirmação é exibido em Brasília. [API e recuperação](../spec.md#5-contratos-da-api)
