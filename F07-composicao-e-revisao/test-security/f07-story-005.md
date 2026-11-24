# Verificação defensiva da gravação local

O recorte abrangeu solicitação de dispositivos, limites, retenção local, acesso na janela e liberação de recursos. O ambiente descartável usou dois eventos e trilhas sintéticas. [Revisão técnica](../code-review/f07-story-005.md) · [Aceite](../test-funcional/2026-11-24_19-24-37/2-execucao.md)

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| White box | Gravar negocia codec antes da permissão. Áudio solicita só o microfone. A sessão limpa trilhas, listeners, temporizadores e blocos no descarte. Respostas de permissão ou de validação que chegam após cancelamento perdem o vínculo com a tentativa. | Aprovado |
| Grey box | Permissão recusada preservou o seletor de arquivo. Dispositivo encerrado durante uma tentativa liberou as trilhas restantes e preservou o rascunho válido. Troca e descarte revogaram as respectivas URLs. | Aprovado |
| Black box guiado | O Nginx entregou política de câmera e microfone limitada à própria origem. O navegador observou zero escritas HTTP durante gravação, parada automática, revisão e descarte. A legenda sintética ficou fora do armazenamento e outro evento abriu sem a mídia. | Aprovado |

O pacote gerado do Amplify passou na verificação local de cinco grupos de cabeçalhos. O runner confirmou zero contribuições persistidas e ausência dos nomes sintéticos, da legenda e dos códigos dos eventos nos logs da UI e do serviço. O contrato encaminha somente o arquivo e os metadados necessários à F08. [Contrato](../dev-front/f07-story-005.md) · [MediaStream Recording](https://www.w3.org/TR/mediastream-recording/)
