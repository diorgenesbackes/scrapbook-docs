# Seleção e revisão de vídeo e áudio

Depoimento permite selecionar um vídeo ou um áudio. Momento permite selecionar um vídeo. A janela continua sob a F05 e a jornada reutiliza a legenda, o diálogo de descarte, a identificação local da F06 e a revisão. [Card](https://trello.com/c/XbEzx9C2) · [PRD](../../PRD.md#f07-composição-e-revisão) · [Seleção de fotos](f07-story-002.md)

| Categoria e mídia | Tamanho máximo | Duração máxima | Containers |
| --- | --- | --- | --- |
| Depoimento em vídeo | 500.000.000 bytes | 300 s | MP4, MOV, WebM |
| Momento em vídeo | 500.000.000 bytes | 60 s | MP4, MOV, WebM |
| Depoimento em áudio | 50.000.000 bytes | 300 s | MP3, M4A, WAV, OGG, WebM |

Os limites em MB usam a unidade decimal já adotada para fotos. A sondagem verifica tamanho antes da leitura, identifica o container por cabeçalho e pede ao navegador metadados e dados decodificáveis. A duração deve ser positiva, finita e ficar no limite inclusive. Áudio e vídeo são distinguidos pela presença de uma faixa de vídeo. Formatos reconhecidos dependem da capacidade nativa de prévia, verificada com amostras. Um arquivo vazio, ilegível ou incompatível recebe orientação de troca. A sondagem sem resposta oferece recuperação após 15 s.

A reprodução exige ação no player nativo e usa pré-carregamento de metadados. Falha posterior da prévia preserva arquivo e legenda e oferece nova tentativa. A seleção mantém o arquivo anterior até validar a substituição. Trocar de etapa, remover, descartar ou sair para a página do evento interrompe a reprodução e libera a fonte. O Nginx e o artefato do Amplify permitem `blob:` em `media-src` e mantêm as regras de imagens da história anterior. [S21](../../design-system/screens/S21.md) · [S22](../../design-system/screens/S22.md) · [S25](../../design-system/screens/S25.md)

## Contrato para a F08

O payload revisado contém `eventCode`, `category`, `format` (`audio` ou `video`), `caption` opcional e `media` com `File` original, `container`, `sizeBytes` e `durationSeconds`. A identificação confirmada é agregada somente para o mesmo evento. As URLs locais são usadas pela interface. A F08 validará os dados recebidos e fará o envio e a confirmação. [Modelagem e mídias](../../guidelines/mensageria.md)
