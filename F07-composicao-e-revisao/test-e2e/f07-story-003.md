# Navegador e integração da seleção de mídia

Em 24/11/2026, `python3 -B media_stack.py` terminou com código 0 em 135 s sobre a UI `93e4b9c`, os testes `45a0ff9` e o serviço `76fd9e0`. A stack descartável aplicou Liquibase sobre PostgreSQL real e usou dois eventos e relógio controlado. A consulta da janela passou pela API da F05.

Playwright 1.65.0 aprovou 20 cenários de vídeo e áudio, 20 regressões de fotos, 20 de texto e 15 de página pública e compartilhamento. Os cinco perfis executaram as suítes sem falhas ou skips. A verificação de desempenho executou um cenário adicional.

| Container e codec da amostra | Chromium 159 | Firefox 159 | WebKit 27.2 | Android emulado | iPhone emulado |
| --- | --- | --- | --- | --- | --- |
| MP4 / H.264 | Prévia | Prévia | Prévia | Prévia | Prévia |
| MOV / H.264 | Prévia | Prévia | Prévia | Prévia | Prévia |
| WebM / VP8 | Prévia | Prévia | Prévia | Prévia | Prévia |
| MP3 / Layer III | Prévia | Prévia | Prévia | Prévia | Prévia |
| M4A / AAC | Prévia | Prévia | Prévia | Prévia | Prévia |
| WAV / PCM | Prévia | Prévia | Prévia | Prévia | Prévia |
| OGG / Vorbis | Prévia | Prévia | Prévia | Prévia | Prévia |
| WebM / Opus | Prévia | Prévia | Prévia | Prévia | Prévia |

A matriz usa mídias sintéticas e confirma a disponibilidade da prévia nativa para as combinações exercitadas. A jornada também confirmou a interrupção da reprodução real ao voltar à edição. A verificação em aparelhos físicos e com outros perfis de codec fica para um ensaio com essas amostras. [Contrato](../dev-front/f07-story-003.md) · [Aceite](../test-funcional/2026-11-24_17-50-00/2-execucao.md)
