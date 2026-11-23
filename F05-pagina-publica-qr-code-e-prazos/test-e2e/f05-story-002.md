# Testes de navegador da F05-STORY-002

`python3 -B event_public_stack.py --browser --load` executou a stack descartável e os 15 cenários da F05 em Chromium, Firefox, WebKit, Android emulado e iPhone emulado. Os cenários percorrem a entrada por teclado, cópia pela Clipboard API e cópia manual, além de testar os três estados da página pública e a recuperação de falha transitória.

O `jsqr` decodifica a prévia SVG na tela, o PNG e o SVG efetivamente baixados. A suíte confere a dimensão do PNG, a chamada de impressão e a prévia de 60 mm com título e URL. O endereço decodificado abre a página sem sessão. O QR permanece idêntico nos estados anterior, aberto e encerrado. [Aceite funcional](../test-funcional/f05-story-002.md)

Em `dev`, a matriz repetiu os 15 cenários com sucesso. `event_stack.py --browser` passou nos 25 cenários de administração. `event_access_stack.py --browser` passou nos cinco cenários de acesso às lembranças. As três execuções também passaram nos respectivos ensaios HTTP e PostgreSQL.

O ensaio de papel com as câmeras de Android e iPhone continua pendente. A matriz emulada verifica os navegadores e a decodificação digital. O item T11 fica aberto até o ensaio físico. [Card](https://trello.com/c/ZbToV6J7)
