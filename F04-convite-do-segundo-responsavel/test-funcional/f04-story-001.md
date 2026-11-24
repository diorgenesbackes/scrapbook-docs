# Aceite funcional local da emissão de convite

| Critério do card | Evidência |
| --- | --- |
| CA1 | HTTP/PostgreSQL confirmam emissão por responsável vinculado, prazo de 24 horas, 401 sem identidade e 404 igual para evento inexistente ou sem vínculo. |
| CA2 | O mesmo pedido retorna o mesmo convite e vencimento. O navegador confere reexibição após recarga, confirmação explícita antes da troca e QR novo. Seis pedidos concorrentes com chaves distintas deixam um único pendente. |
| CA3 | A API oculta o convite e recusa emissão se o evento estiver travado ou lotado. O pendente deixa de ser devolvido no instante exato do vencimento. |
| CA4 | O teste unitário confere origem canônica com alias. Playwright decodifica os QR, opera os botões por teclado, verifica seleção manual e abre a entrada em outro contexto sem consumir o convite. |
| CA5 | As respostas têm `no-store`. O teste defensivo confirma que os logs e a auditoria omitem a credencial. Transições terminais removem o segredo recuperável. |

O ensaio usa contas sintéticas e relógio controlado. A futura publicação precisa conferir os headers e chaves no ambiente real. [E2E](../test-e2e/f04-story-001.md) · [Segurança](../test-security/f04-story-001.md)
