# Plano de aceite da seleção de vídeo e áudio

Validar CA1–CA4 em Docker com duas páginas públicas, relógio controlado, identificação sintética e arquivos de mídia próprios. [Card](https://trello.com/c/XbEzx9C2) · [PRD](../../../PRD.md#f07-composição-e-revisão) · [Contrato](../../dev-front/f07-story-003.md)

| Critério | Cenário e resultado esperado |
| --- | --- |
| CA1 | Depoimento oferece vídeo e áudio. Momento oferece vídeo, fotos e apenas os formatos da categoria. Os limites de duração aceitam 60 e 300 s quando correspondentes. A verificação unitária cobre a borda de bytes. |
| CA2 | Amostras reais verificam MP4, MOV, WebM, MP3, M4A, WAV e OGG. Arquivo vazio, corrompido ou com faixa incompatível bloqueia Revisar e indica recuperação. |
| CA3 | A revisão apresenta reprodução explícita, legenda e ações para trocar, remover ou voltar. O contrato preserva arquivo, container, tamanho, duração e código. |
| CA4 | Falha de prévia conserva legenda e permite retentar. Edição, troca, descarte e saída encerram o player ou liberam a URL aplicável. Uma página de outro evento inicia vazia. |

Medir metadados e prévias de MP4/H.264, WebM/VP8 e WAV/PCM em perfil móvel. Repetir regressões de texto, fotos e página pública e verificar a ausência de escrita HTTP e de dados de rascunho na telemetria.
