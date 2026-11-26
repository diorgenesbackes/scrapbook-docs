# Revisão técnica da F08-STORY-005

A revisão independente comparou o serviço `b300642` com `224c469`, a interface `b34aa71` com `5c9a7dc` e os testes `1593873` com `2a9dffa`. Conferiu o contrato da tentativa, a fixação da versão, a concorrência entre recepção e validação, a cópia privada e o recebimento do recibo. A fixação da prova e a liberação da reserva de validação ocorrem na mesma transação. O teste HTTP cobriu o limite com precisão de milissegundo. O corte a 100 ns após o limite foi verificado em teste unitário. A concorrência entre duas instâncias foi examinada no SQL. [Spec](../spec.md) · [Cartão](https://trello.com/c/KrtTW39U)

| Achado | Correção | Reteste |
| --- | --- | --- |
| P2. A duração medida por faixa deixava de fora os intervalos iniciais de M4A e as defasagens entre faixas. | Medir a linha do tempo comum da mídia, preservando a duração por amostras e o ajuste de quantização WebM. | O M4A com duas faixas defasadas passou de 2,027 s para 307,005 s. M4A com intervalo inicial mediu 300 s no limite e 300,001 s acima. |

O reteste com FFmpeg 9.1 mediu exatamente 300 s para MP3, M4A, WAV, Ogg e WebM e identificou o excesso nas cinco amostras superiores a 300 s. Também conferiu WAV e M4A de 44,1 kHz, Ogg Vorbis e FLAC, capa de MP3 e WebM de gravação. O runner HTTP ganhou uma regressão para o M4A com duas faixas.

| Verificação local | Resultado |
| --- | --- |
| SDK .NET 10.0.401, restore travado em cache local com auditoria de rede desativada | Sucesso |
| Build do serviço | Sucesso, zero erros e avisos |
| Testes xUnit | 285 aprovados, 32 casos novos de áudio, zero pulados |
| `npm run check` | Lint e build aprovados. 181 testes Vitest aprovados, 10 casos novos de áudio |
| Descoberta no Playwright 1.65.0 | Sete casos de áudio |

Sem achados remanescentes no escopo revisado. A rodada final da integração usa os SHAs mesclados em `dev`.
