# Navegador e integração da gravação

Em 24/11/2026, `python3 -B recording_stack.py` terminou com código 0 em 195 s sobre a UI `6156565`, os testes `948bc5a` e o serviço `76fd9e0`. A stack Docker usou PostgreSQL com Liquibase, Nginx, dois eventos sintéticos e a janela real da F05. O runner usou Playwright 1.65.0.

| Perfil de gravação | Vídeo | Áudio | Cenários |
| --- | --- | --- | --- |
| Chromium 158.0.8127.18 desktop | MP4 com H.264 e AAC | M4A com AAC | 6 |
| Firefox 159.0 desktop | WebM com VP8 e Opus | Ogg com Opus | 6 |
| Chromium 158.0.8127.18 com Android emulado | MP4 com H.264 e AAC | M4A com AAC | 6 |

As gravações usaram dispositivos sintéticos do navegador. Os cenários de parada em 60 e 300 s avançaram o relógio da página e verificaram a decodificação de uma gravação nativa curta. O ensaio de 4G adicional gravou um minuto real. A recusa de microfone foi aplicada pela política HTTP em Chromium e por uma resposta sintética em Firefox.

Passaram 85 regressões, incluindo 10 de câmera fotográfica, 20 de seleção de fotos, 20 de seleção de vídeo e áudio, 20 de texto e 15 de página pública. As regressões de seleção de mídia, texto e página pública também testaram WebKit desktop e iPhone emulado. O axe reportou zero violações no escopo dos controles de gravação. Não houve skips. [Aceite](../test-funcional/2026-11-24_19-24-37/2-execucao.md) · [Desempenho](../test-performance/f07-story-005.md) · [Segurança](../test-security/f07-story-005.md)

Um ensaio avulso em WebKit 27.2 exerceu o mesmo editor em desktop e iPhone emulado. Com trilhas fornecidas por canvas e oscilador, os dois perfis produziram vídeo MP4 com H.264 e AAC e áudio M4A com AAC. As quatro saídas abriram a revisão com duração positiva e trilhas encerradas. A configuração inicial esperava a ativação do oscilador antes de devolver a trilha e expirou no áudio. O reteste isolou os contextos e conferiu a ativação do oscilador separadamente. Permissão nativa e captação em dispositivos físicos ficam para homologação.
