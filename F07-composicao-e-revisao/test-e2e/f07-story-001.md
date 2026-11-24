# Navegador e integração do depoimento em texto

Em 24/11/2026, `python3 -B testimonial_stack.py` terminou com código 0 sobre a UI `43c3862`, os testes `142c9fc` e o serviço `76fd9e0`. O laboratório aplicou Liquibase em PostgreSQL 18.6, preparou dois eventos sintéticos com relógio controlado e usou a API real da F05.

Playwright 1.65.0 passou 20 cenários da história e 15 regressões da página pública e compartilhamento, em Chromium 159.0.8154.6, Firefox 159.0, WebKit 27.2, Android emulado e iPhone emulado. Foram verificados janela, rota direta, limite, colagem Unicode, teclado, descarte, isolamento entre eventos e identificação com aviso sintético. A revisão a 320 px passou na verificação automatizada axe do recorte. Aparelhos físicos e leitores de tela ficam para validação de compatibilidade.

O primeiro ciclo da branch passou em 19 de 20 cenários. O teste Firefox passou a fornecer explicitamente os dados do evento sintético de colagem, preservando as asserções de limite e sufixo. A repetição da branch e o ensaio integrado passaram em 20 de 20. [Revisão](../code-review/f07-story-001.md) · [Aceite](../test-funcional/2026-11-24_13-48-27/2-execucao.md)
