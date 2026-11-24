# Navegador e integração da primeira identificação

Em 24/11/2026, `python3 -B guest_identification_stack.py` terminou com código 0 sobre o frontend `93d0bf8`, os testes `455c07d` e o serviço `76fd9e0`. O laboratório usou Docker 29.8.2, PostgreSQL 18.6 com migrations do Liquibase, relógio controlado e dois eventos sintéticos abertos. As consultas de página pública foram reais.

Playwright 1.65.0 executou três casos em cada um dos cinco perfis, com 15 aprovações, zero falhas e zero skips. A matriz incluiu Chromium 159.0.8154.6, Firefox 159.0, WebKit 27.2, Android emulado e iPhone emulado. Foram verificados identificação por teclado, foco, mensagens, aviso, retorno ao evento, troca de contexto, link desconhecido e layout a 320 px com axe. O perfil desktop de Chromium também percorreu a jornada a 1280 px. Leitores de tela e aparelhos físicos ficaram fora desta execução.

A unidade `GuestIdentificationForm` confirmou a entrega do código do evento atual, nome normalizado, versão do aviso e instante ao consumidor. O ensaio integrado confirmou somente a etapa da F06. O fluxo de envio da contribuição pertence à F08. [Contrato](../dev-front/f06-story-001.md) · [Revisão](../code-review/f06-story-001.md)
