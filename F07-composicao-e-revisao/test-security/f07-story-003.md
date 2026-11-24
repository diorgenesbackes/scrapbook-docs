# Verificação defensiva da seleção de vídeo e áudio

O escopo cobriu a classificação, os limites, a decodificação local, a política de conteúdo e o isolamento do rascunho. [Revisão técnica](../code-review/f07-story-003.md) · [Candidato exercitado](../test-e2e/f07-story-003.md)

| Perspectiva | Evidência | Resultado |
| --- | --- | --- |
| White box | Os controles locais inspecionam o prefixo, a faixa, a duração e a prontidão do player. A seleção de arquivos não solicita câmera ou microfone. As cinco regras geradas pelo pacote do Amplify e a configuração Nginx admitem `blob:` em `media-src` e `img-src`. | Aprovado |
| Grey box | Arquivos vazios, corrompidos, com prefixo incompatível ou duração acima do limite bloquearam a revisão. A tentativa inválida de substituição preservou a seleção válida e a legenda. Voltar pausou a reprodução. Troca, remoção e descarte liberaram as URLs observadas. | Aprovado |
| Black box guiado | As jornadas observaram zero escritas HTTP, isolamento entre dois eventos e ausência dos nomes e da legenda sintéticos no armazenamento e na telemetria verificados. A consulta ao banco descartável registrou zero contribuições. | Aprovado |

O contrato local segue para a validação autoritativa da F08 quando houver recebimento de objetos. [Contrato](../dev-front/f07-story-003.md)
