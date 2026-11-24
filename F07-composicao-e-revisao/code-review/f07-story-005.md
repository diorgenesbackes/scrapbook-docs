# Revisão técnica da F07-STORY-005

O recorte percorreu o editor, a negociação de codec, a gravação, os limites, a integração com a seleção e a liberação de recursos. O serviço usado na verificação está em `76fd9e0`. [Contrato](../dev-front/f07-story-005.md) · [Base de mídias](../dev-front/f07-story-003.md)

- A capacidade de gravar e reproduzir é verificada antes do pedido de dispositivo. As respostas tardias são descartadas com parada das trilhas. O gravador mantém blocos locais dentro do limite, encerra as trilhas na parada e aguarda o arquivo com timeout técnico.
- A integração usa o detector e o player existentes. O WebM gravado com duração inicialmente infinita passa por uma leitura do fim no próprio navegador. A revisão exige tamanho válido e duração finita no arquivo decodificado. O cancelamento aborta a sonda sem sobrescrever uma seleção anterior.
- A política gerada no Nginx e no Amplify permite câmera e microfone para a própria origem. Nos cinco grupos do pacote estático, a geolocalização fica bloqueada e o documento não pode ser incorporado em frames.

O primeiro teste dirigido aguardava blocos intermediários de vídeo. Chromium pôde aguardar a parada antes de entregar o primeiro bloco útil. A jornada passou a aguardar quadros na prévia e validar os bytes no arquivo final. O reteste expôs no Firefox uma trilha encerrada sem o evento esperado. O estado das trilhas passou a ser conferido também na atualização do contador e antes da parada. O reteste seguinte passou nos três perfis e verificou a recuperação com a mídia anterior.

| Check | Resultado |
| --- | --- |
| Oxlint, TypeScript, Vite e Vitest | Exit 0 e 118 unitários aprovados. A saída do lint e do build não exibiu warnings. |
| Restore bloqueado, compilação limpa e unitários .NET | Exit 0, zero warnings de compilação e 125 unitários aprovados. |
| Reteste dirigido em Docker | 18 cenários aprovados de gravação, revisão e recuperação. |
| `git diff --check` e pacote local Amplify | Exit 0 e cinco grupos de cabeçalhos verificados. |

O ciclo unitário não coletou percentuais de cobertura. Sem achados técnicos remanescentes no recorte revisado.
