# F03-STORY-004 · Validação ponta a ponta

`python3 -B event_access_stack.py --browser --load` aprovou cinco cenários em Chromium, Firefox e WebKit desktop, Android e iPhone emulados. O roteiro abre a área pelo teclado, lê data e contador, verifica ausência de imagens e nome do evento e passa à confirmação na virada da meia-noite. Uma alteração persistida de imediato para futuro recupera a tela fechada. A infraestrutura da execução tem o relógio da aplicação controlado por fixture e nenhum serviço de avisos.

`python3 -B event_stack.py --browser` aprovou os 25 cenários de regressão de criação, edição e política de abertura na imagem normal do serviço.

[Validação funcional](../test-funcional/f03-story-004.md) · [Evidências de entrega](../delivery/f03-story-004.md)
