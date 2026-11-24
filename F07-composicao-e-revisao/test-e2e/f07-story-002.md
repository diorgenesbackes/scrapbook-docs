# Navegador e integração da seleção de fotos

Em 24/11/2026, `python3 -B photo_stack.py` terminou com código 0 sobre a UI `9c7aa7c`, os testes `8c8dfcf` e o serviço `76fd9e0`. A stack descartável aplicou Liquibase sobre PostgreSQL 18.6 e usou dois eventos e o relógio controlado. A consulta da janela usou a API da F05.

Playwright 1.65.0 aprovou 20 cenários de fotos, 20 regressões de texto e 15 regressões de página pública e compartilhamento. Houve cinco perfis, sem falhas ou skips. As jornadas cobriram seleção, validação, ordem, troca, remoção, legenda, identificação e descarte. A revisão a 320 px passou nas verificações automatizadas do axe no escopo.

| Perfil | Prévia da amostra HEIC |
| --- | --- |
| Chromium 159.0.8154.6 e Android emulado | Orientação de recuperação com a seleção válida preservada. |
| Firefox 159.0 | Orientação de recuperação com a seleção válida preservada. |
| WebKit 27.2 e iPhone emulado | Prévia decodificada. |

O teste usou uma amostra HEIC sintética de 32 × 32. A classificação de marcas HEIF adicionais foi coberta em unitários. A compatibilidade com aparelhos físicos e outros perfis HEIF requer amostras e dispositivos próprios. [Contrato](../dev-front/f07-story-002.md) · [Aceite](../test-funcional/2026-11-24_14-12-15/2-execucao.md)
