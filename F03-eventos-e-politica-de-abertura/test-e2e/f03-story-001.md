# F03-STORY-001 · Testes ponta a ponta

`python3 -B event_stack.py --browser --load` exercitou o candidato integrado em `dev` no Docker descartável. A jornada navegou pela segunda página de eventos, abriu o formulário pelo teclado, conferiu a sugestão de 28/02 e salvou com data ajustada. O Playwright enviou a primeira solicitação para a API e interrompeu a entrega da resposta ao navegador. A recuperação usou a mesma chave, recebeu `200` e abriu o detalhe com os valores persistidos.

Passaram os cinco casos, em Chromium, Firefox e WebKit desktop, Android emulado e iPhone emulado. O ensaio usa contas sintéticas e credenciais restritas ao loopback. Leitores de tela e dispositivos físicos ficam para a matriz de compatibilidade do produto. [Validação funcional](../test-funcional/f03-story-001.md)
