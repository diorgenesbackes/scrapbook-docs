# Aceite do convite no navegador

`python3 -B invitation_acceptance_stack.py --browser --load` executou cinco cenários em Chromium, Firefox, WebKit, Android emulado e iPhone emulado. O emissor gerou o QR por teclado. O segundo contexto, com outra conta, abriu o endereço decodificado. A prévia removeu o fragmento da barra de endereço e exibiu o nome do evento. Antes de clicar em “Aceitar convite”, a conta recebeu 404 ao abrir o detalhe.

O teste descartou a primeira resposta depois de o serviço confirmar o aceite. “Recuperar aceite” devolveu o mesmo evento, abriu o detalhe e permitiu encontrá-lo na listagem “Meus eventos”. A matriz usa credenciais sintéticas. [Roteiro de aceite](../test-funcional/f04-story-002.md)

Em `dev`, passaram os cinco cenários de aceite, os cinco cenários de emissão e 25 cenários de criação, edição e abertura da F03. As três suítes passaram nos respectivos ensaios de HTTP e PostgreSQL.
