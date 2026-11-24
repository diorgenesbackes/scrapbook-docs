# Verificação defensiva da captura fotográfica

O recorte avaliou pedido de permissão, ciclo de vida da câmera e tratamento do JPEG e da legenda. O laboratório usou dois eventos e uma câmera sintética local. [Revisão técnica](../code-review/f07-story-004.md) · [Aceite](../test-funcional/2026-11-24_19-05-34/2-execucao.md)

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| White box | O pedido de dispositivo está nos eventos Fotografar e Refazer, com `audio: false`. O quadro é convertido localmente, as trilhas são encerradas antes de aguardar o JPEG e as respostas tardias são descartadas. O contrato de revisão leva o arquivo preparado, sem URL de apresentação. | Aprovado |
| Grey box | A jornada mediu zero pedidos ao abrir o editor. Com acesso recusado, o navegador retornou `NotAllowedError` e a rota permitiu continuar por arquivo. As trilhas foram encerradas ao fotografar ou sair da captura. As URLs das fotos substituídas ou descartadas foram revogadas. | Aprovado |
| Black box guiado | O Nginx serviu a política de câmera para a própria origem com microfone e geolocalização negados. O navegador registrou zero escritas HTTP e nenhum resíduo de arquivo ou legenda no armazenamento observado. O segundo evento não herdou o rascunho. | Aprovado |

A suíte encontrou zero contribuições no banco descartável e conferiu a ausência de nome de arquivo, legenda e códigos de evento nos logs da UI e do serviço. A validação autoritativa de recebimento integra a F08. A política do Amplify foi conferida no pacote gerado localmente. [Contrato](../dev-front/f07-story-004.md) · [Media Capture and Streams](https://www.w3.org/TR/mediacapture-streams/)
