# Execução — identidade visual

Cartão [Create the project’s visual identity](https://trello.com/c/0YAjwM8m), feature/3 em docs/tests. Executor Codex. Direção explicitamente escolhida pelo responsável: **fotográfico minimalista**, fundos neutros, poucos adornos e destaque para mídias. Proposta 0.1; revisão de todas as telas e decisões Q01–Q08 ainda pendentes, sem aceite humano presumido.

## Entrega

[Guia](../../visual-identity/README.md) e catálogo HTML local: marca/símbolo/favicon originais em SVG, fotografia sintética, paleta HEX, tipografia/fallbacks, escala em rem/px, medidas/radius/bordas/sombras/camadas, componentes e estados interativos, responsivo/acessibilidade, snippets HTML/CSS e modelo de e-mail inline. Inventário de **38 superfícies e 178 estados**, rastreando FR-001–FR-022, com especificações individuais e destinos de navegação.

Telas de conta, responsáveis/eventos/convites, convidados/captura, revisão/envio/recibo, bloqueio/galeria/player e mensagens operacionais estão cobertas. Considerar confirmação de e-mail, recuperação por código de uma hora e login gerenciado conforme adendos aprovados do HLD. Não implementar autenticação própria a partir dos mocks. FR-020/022 não viraram painel administrativo fora do PRD.

Três fotos sintéticas foram produzidas pela skill imagegen/ferramenta integrada, inspecionadas e copiadas para o projeto. [Ativos, origem e prompts completos](../../visual-identity/assets/README.md). Fontes vêm do sistema, sem arquivos externos ou bibliotecas novas. Os PNGs originais somam 6,64MiB e são referência; handoff exige otimização/variantes antes do uso em produção, sem afirmar o objetivo 4G de três segundos.

## Comandos e resultados

`node --check` do catálogo, referências locais e `git diff --check` aprovados. Cálculo de **20 pares de contraste** aprovado, incluindo texto normal ≥4.5:1 e limites/foco ≥3:1; menor par de texto normal 4.73:1. [Dados calculados](../../visual-identity/contrast.json).

`npx playwright test --config visual-identity.config.ts`: **14 testes aprovados**, zero falhas/ignorados/flaky, duração final 73,414s. Início UTC: 2026-11-13T16:07:49.492Z. Chromium 158.0.8127.18 e WebKit 27.2, com contextos desktop/móvel emulado. **712 verificações de estado** (178×2 larguras×2 browsers), em 320/1440px, sem overflow horizontal ou assets quebrados. Telas representativas também em 390/768px; proporções de fotos/hero verificadas. Hierarquia de títulos, foco visível, abas/setas/Home/End, diálogo/Escape/retorno, skip link, texto 200% e reduced motion verificados.

Bloqueio e recibo não montam fotos/players/autores de demonstração. Alterar nome limpa concordância. Contadores aparecem nos limiares definidos, limitam entrada preservando Unicode e permitem editar/apagar. Isso valida apenas as composições e interações do protótipo; não é prova de autorização do backend ou fluxo de envio real.

## Inspeção visual e correções

A primeira rodada teve 9 aprovações/5 falhas: ciclo de foco do diálogo, segundo h1 usado como amostra tipográfica e espera por imagens lazy fora da tela. Corrigidos foco cíclico explícito e semântica do exemplo; a suíte passou a tornar imagens da captura eager e esperar decodificação/carregamento. Uma execução intermediária, com 13 aprovações, foi interrompida porque a captura WebKit aguardava imagens lazy; não conta como aprovação integral.

A inspeção manual de capturas revelou altura intrínseca fixa nas imagens apesar do aspect-ratio. Aplicado height:auto e adicionada verificação de proporções para hero/mobile/cards. Contador visual deixou de anunciar cada tecla e usa avisos por marco. A rodada final acima aprovou. Warning NO_COLOR/FORCE_COLOR não alterou resultado.

Capturas finais inspecionadas: [visão geral desktop](overview-desktop.png), [convidado mobile](guest-mobile.png), [galeria desktop](gallery-desktop.png). Conteúdo e pessoas são sintéticos. [Resumo sanitizado](resultado.json).

## Limites e revisão pendente

Não foram realizados testes com pessoas, leitor de tela, aparelhos físicos, matriz completa de navegadores, clientes de e-mail, Cognito, câmera/microfone, QR escaneável, envio/reprodução reais, 4G ou carga funcional. Não se declara conformidade integral WCAG ou aceite do produto. [Questões Q01–Q08](../../visual-identity/decisions.md) registram lacunas de provedor, capa, privacidade, domínio/suporte, usabilidade, rotas inferidas, aprovação visual e contagem Unicode.

Backend, UI funcional, infraestrutura, banco e migrations não foram alterados. Portas 5173/5080/55432 permanecem fechadas. O runner encerrou seu servidor exclusivo em 4175. A prévia **somente do catálogo** em http://127.0.0.1:4174 foi mantida para revisão, servindo apenas visual-identity/.

## Dependência e rastreabilidade

Feature/3 parte da feature/6 já entregue, ainda não promovida: docs e475c68897902cacc7c68e3e64aae560b1044f35; tests b8c8f4a2b2b51f469bf8a2c56774bae8d3b08092. Sem merge/deploy/publicação. Transições reais UTC e evidências por task são registradas abaixo, sem inferir horas ativas ou aprovação humana.

## Revisões validadas

- scrapbook-docs: [fe121cc27b466c9a4207eb3b575d17a21357e8f3](https://github.com/diorgenesbackes/scrapbook-docs/commit/fe121cc27b466c9a4207eb3b575d17a21357e8f3).
- scrapbook-tests: [b595bdd7290cac4c19eb1338c80929748a0f458a](https://github.com/diorgenesbackes/scrapbook-tests/commit/b595bdd7290cac4c19eb1338c80929748a0f458a).

Evidências e transições: este relatório e seu commit posterior na mesma feature.

| Task | Primeiro Doing UTC | Último Test UTC | To Deploy UTC |
| --- | --- | --- | --- |
| [VI-001 — Inventariar telas e estados do PRD](https://trello.com/c/vNIDkZGb/153-vi-001-inventariar-telas-e-estados-do-prd) | 2026-11-13T14:58:00.415Z | 2026-11-13T16:00:14.635Z | 2026-11-13T16:11:36.568Z |
| [VI-002 — Definir marca e ativos vetoriais](https://trello.com/c/hE0SAWgP/154-vi-002-definir-marca-e-ativos-vetoriais) | 2026-11-13T15:01:19.580Z | 2026-11-13T16:00:16.195Z | 2026-11-13T16:11:39.392Z |
| [VI-003 — Definir tokens visuais e contraste](https://trello.com/c/ELBduslD/155-vi-003-definir-tokens-visuais-e-contraste) | 2026-11-13T15:01:20.683Z | 2026-11-13T16:07:14.386Z | 2026-11-13T16:11:41.981Z |
| [VI-004 — Especificar componentes e estados](https://trello.com/c/wLBIdvHK/156-vi-004-especificar-componentes-e-estados) | 2026-11-13T15:01:21.845Z | 2026-11-13T16:02:33.108Z | 2026-11-13T16:11:44.895Z |
| [VI-005 — Especificar navegação e responsividade](https://trello.com/c/Tdlkf4sy/157-vi-005-especificar-navega%C3%A7%C3%A3o-e-responsividade) | 2026-11-13T15:01:23.406Z | 2026-11-13T16:00:20.869Z | 2026-11-13T16:11:47.692Z |
| [VI-006 — Especificar telas de conta e autenticação gerenciada](https://trello.com/c/hHqYNair/158-vi-006-especificar-telas-de-conta-e-autentica%C3%A7%C3%A3o-gerenciada) | 2026-11-13T15:01:24.718Z | 2026-11-13T16:00:22.174Z | 2026-11-13T16:11:50.807Z |
| [VI-007 — Especificar eventos, responsáveis e convites](https://trello.com/c/4oPuhnMm/159-vi-007-especificar-eventos-respons%C3%A1veis-e-convites) | 2026-11-13T15:01:25.966Z | 2026-11-13T16:00:23.603Z | 2026-11-13T16:11:53.621Z |
| [VI-008 — Especificar participação e captura](https://trello.com/c/y6E5Um8C/160-vi-008-especificar-participa%C3%A7%C3%A3o-e-captura) | 2026-11-13T15:01:27.267Z | 2026-11-13T16:00:25.339Z | 2026-11-13T16:11:56.631Z |
| [VI-009 — Especificar revisão e estados de envio](https://trello.com/c/HJJ6MXFu/161-vi-009-especificar-revis%C3%A3o-e-estados-de-envio) | 2026-11-13T15:01:28.847Z | 2026-11-13T16:07:15.726Z | 2026-11-13T16:11:59.109Z |
| [VI-010 — Especificar bloqueio, galeria e reprodução](https://trello.com/c/5gXJ447L/162-vi-010-especificar-bloqueio-galeria-e-reprodu%C3%A7%C3%A3o) | 2026-11-13T15:01:30.024Z | 2026-11-13T16:00:28.850Z | 2026-11-13T16:12:01.903Z |
| [VI-011 — Especificar e-mails e estados operacionais](https://trello.com/c/PbvTwL6P/163-vi-011-especificar-e-mails-e-estados-operacionais) | 2026-11-13T15:01:31.085Z | 2026-11-13T16:00:30.672Z | 2026-11-13T16:12:06.098Z |
| [VI-012 — Construir catálogo HTML/CSS navegável](https://trello.com/c/NpTeZOQ0/164-vi-012-construir-cat%C3%A1logo-html-css-naveg%C3%A1vel) | 2026-11-13T15:01:32.505Z | 2026-11-13T16:07:16.886Z | 2026-11-13T16:12:09.576Z |
| [VI-013 — Validar catálogo e acessibilidade visual](https://trello.com/c/3KakQWCw/165-vi-013-validar-cat%C3%A1logo-e-acessibilidade-visual) | 2026-11-13T15:58:14.026Z | 2026-11-13T16:07:18.059Z | 2026-11-13T16:12:13.185Z |
| [VI-014 — Consolidar guia e decisões para revisão](https://trello.com/c/mf0EGV0y/166-vi-014-consolidar-guia-e-decis%C3%B5es-para-revis%C3%A3o) | 2026-11-13T15:01:34.351Z | 2026-11-13T16:00:35.660Z | 2026-11-13T16:12:17.172Z |

[Transições completas](transicoes.json). Cartão e 14 tasks em To Deploy, com revisão visual humana pendente. A direção geral foi escolhida; não se presume aprovação integral da proposta.
