# Jornada da conta convidada

A S14 lê a credencial no fragmento, remove o fragmento do endereço e mantém o token em memória. A prévia exibe contexto somente após autenticação e confirmação de e-mail. O botão “Aceitar convite” inicia a escrita. Ao confirmar, a interface abre o detalhe do evento, disponível também em “Meus eventos”.

Se a resposta do aceite se perdeu, a ação “Recuperar aceite” repete a operação com o mesmo token. A tela anuncia carregamento e falhas, com mensagens para expiração, substituição, uso anterior, lotação, trava e conta sem confirmação. Os cenários locais usam identidades sintéticas. A retomada depois de entrar ou cadastrar uma conta pertence à integração F02. [Design system S14](../../design-system/screens/S14.md)
