# Jornada de navegador da emissão de convite

`python3 -B invitation_stack.py --browser --load` executa a jornada em Chromium, Firefox, WebKit, Android emulado e iPhone emulado. Os cinco cenários geraram o convite por teclado, decodificaram o QR com `jsqr`, copiaram por seleção manual, abriram o endereço em um contexto novo e confirmaram a remoção do fragmento. O convite permaneceu pendente. Recarregar manteve o endereço, cancelar preservou o convite, e confirmar a substituição produziu um QR novo.

A tela de entrada oferece orientação neutra. O aceite pela pessoa convidada será exercitado ponta a ponta na segunda story. [Card de aceite](https://trello.com/c/QW1wB9y2)
