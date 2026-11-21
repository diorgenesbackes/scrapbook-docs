# F03-STORY-004 · Desenvolvimento de frontend

O detalhe oferece o acesso a `/meus-eventos/{id}/lembrancas`. A área fechada segue a S28 com data civil, contador em dias, horas e minutos e status neutro durante a confirmação. O contador usa a diferença entre os instantes de abertura e do servidor, atualizada com o relógio monotônico do navegador. A área passa a aberta depois de uma resposta autorizada. A galeria será integrada pela F11.

A rota é remontada ao trocar de evento. As consultas anteriores são canceladas ao iniciar uma nova, ao sair da página ou ao voltar de uma sessão em segundo plano. Uma alteração de política em uma aba sinaliza às outras abas para consultar o servidor, e uma consulta periódica de 15 segundos cobre alterações externas à aba. Respostas com 401 ou 404 substituem o estado anterior por feedback seguro.

[S28](../../design-system/screens/S28.md) · [Contrato de estado](../spec.md#estado-de-abertura-e-autorização-da-galeria)
