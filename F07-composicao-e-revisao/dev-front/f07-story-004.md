# Captura fotográfica da F07-STORY-004

No editor de fotos, Fotografar abre a prévia da câmera por ação explícita. Depoimento comporta uma foto e momento comporta até dez. A seleção de arquivos permanece disponível. Refazer abre a câmera para a posição escolhida e conserva a foto anterior até que a substituta passe pela validação. [Card](https://trello.com/c/uxlzqFB7) · [S20](../../design-system/screens/S20.md) · [Contrato de fotos](f07-story-002.md)

A captura pede apenas vídeo, com preferência pela câmera traseira. O quadro é convertido em JPEG com qualidade 0,9. O lado maior fica limitado a 1.920 px, com proporção preservada. O arquivo passa pela mesma validação de 20.000.000 bytes, assinatura e decodificação da seleção de fotos. A legenda e o contrato de revisão são compartilhados com os demais arquivos aceitos. [PRD](../../PRD.md#f07-composição-e-revisão) · [Revisão](../../design-system/screens/S25.md)

As trilhas são encerradas assim que o quadro é copiado ou quando a prévia perde o contexto de edição. Voltar aos formatos, trocar para arquivo, abrir o diálogo de saída, fechar a página ou encerrar a janela interrompem a câmera. Se uma solicitação de permissão terminar depois do cancelamento, a trilha retornada é encerrada imediatamente. O navegador pode manter o diálogo de permissão até o convidado responder.

O Nginx e o pacote do Amplify permitem câmera para a própria origem do documento. O navegador ainda exige permissão, e a interface só pede acesso no editor de fotos dentro da janela. Microfone, geolocalização e incorporação por iframe ficam bloqueados pelos cabeçalhos servidos. O arquivo e a legenda permanecem em memória durante a revisão. [Guia de frontend](../../guidelines/frontend.md#7-captura-revisão-e-envio)
