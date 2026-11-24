# Navegador e integração da câmera fotográfica

Em 24/11/2026, `python3 -B camera_stack.py` terminou com código 0 em 124 s. O ensaio usou UI `d05fb32`, testes `97c2e35` e serviço `76fd9e0`. A stack descartável usou PostgreSQL com Liquibase, dois eventos, relógio controlado, a API da F05 e o Nginx da UI.

| Suíte | Cenários aprovados | Perfis |
| --- | --- | --- |
| Câmera fotográfica | 10 | Chromium desktop e Android emulado com dispositivo sintético. |
| Fotos selecionadas | 20 | Chromium, Firefox, WebKit, Android e iPhone emulados. |
| Vídeo e áudio selecionados | 20 | Os mesmos cinco perfis. |
| Texto | 20 | Os mesmos cinco perfis. |
| Página pública e compartilhamento | 15 | Os mesmos cinco perfis. |
| Prontidão de câmera em 4G | 1 cenário com 10 capturas | Android emulado. |

A prévia ativa passou nas regras automatizadas do axe no escopo do editor. A integração confirmou que cada quadro capturado se torna um JPEG decodificável, passa pela seleção comum e aparece na revisão. O runner verificou zero contribuições persistidas e ausência dos dados sintéticos observados nos logs. Não houve skips. [Aceite](../test-funcional/2026-11-24_19-05-34/2-execucao.md) · [Desempenho](../test-performance/f07-story-004.md) · [Segurança](../test-security/f07-story-004.md)

O Chromium testado foi 158.0.8127.18. As regressões também usaram Firefox 159.0 e WebKit 27.2. A câmera física e a captura por Firefox e WebKit ficaram fora do ensaio de câmera.
