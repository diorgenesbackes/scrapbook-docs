# F03-STORY-003 · Validação ponta a ponta

`python3 -B event_stack.py --browser --load` executou 25 cenários com sucesso em Chromium, Firefox e WebKit desktop, Android e iPhone emulados. Dez cenários exercitaram diretamente a story.

1. Criar evento com mensagem personalizada, manter rascunho após edição em outra sessão, consultar a versão concorrente e confirmar novo envio. Conferir os três prazos, restaurar a mensagem padrão e gravar uma nova mensagem personalizada.
2. Abrir um evento cuja trava foi preparada com um registro confirmado sintético. Conferir a mensagem de bloqueio e a independência da configuração de abertura.

O primeiro passe identificou um seletor de texto que correspondia ao resumo e à área de texto. O teste passou a consultar o parágrafo do resumo. O reteste completo passou nas cinco configurações.

[Testes de aceite](../test-funcional/f03-story-003.md) · [Candidato](../delivery/f03-story-003.md)
