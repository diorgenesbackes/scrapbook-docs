# Fundamentos visuais

[Índice](README.md) · [Tokens exatos](tokens.css) · [Prévia](index.html#foundations)

## Marca

Nome público: **Scrap Book**, duas palavras. Símbolo original de duas molduras/páginas sobrepostas, sem câmera literal, corações ou adorno sazonal. A marca funciona em casamentos, aniversários e encontros.

- Assinatura: [brand-lockup.svg](assets/brand-lockup.svg), proporção 252:48; mínimo 128px/8rem de largura. Texto em Georgia/fallback; para impressão final, conferir substituição de fonte no dispositivo ou converter em contornos com fonte licenciada do operador.
- Símbolo: [positivo](assets/brand-symbol.svg) e [inverso](assets/brand-symbol-inverse.svg), viewBox 48×48; mínimo 24px/1.5rem. Favicon dedicado em 16/32/48px pelo SVG próprio.
- Área de proteção: 0.5× a largura do símbolo em todos os lados (12px para símbolo de 24px). Não deformar, girar, sombrear ou recolorir livremente.
- Assinatura positiva em surface/canvas; inversa apenas em ink. Sobre fotografia, colocar marca em superfície neutra opaca. Nunca depender do contraste incerto da foto.

## Paleta e papéis

Todas as cores são sRGB. Nomes CSS têm prefixo --sb-. O fundo ocupa a maior área; cores de estado aparecem em blocos pequenos e sempre com texto/ícone. border é somente decorativo; control-border delimita campos com contraste suficiente. disabled permanece legível e tem explicação próxima.

| Token | HEX | Papel |
| --- | --- | --- |
| --sb-canvas | #F6F5F2 | Fundo geral |
| --sb-surface | #FFFFFF | Conteúdo/cartões |
| --sb-ink | #252823 | Texto principal |
| --sb-muted | #60665E | Texto de apoio |
| --sb-primary | #252823 | Ação principal |
| --sb-primary-hover | #3B4239 | Hover primário |
| --sb-primary-active | #151813 | Pressionado |
| --sb-selected | #EBEEE8 | Seleção/preenchimento neutro |
| --sb-border | #DADCD5 | Divisória decorativa |
| --sb-control-border | #81877C | Limite de campo/controle |
| --sb-focus | #45654C | Anel de foco |
| --sb-success | #2F6448 | Texto/ícone de sucesso |
| --sb-success-bg | #EDF5F0 | Fundo de sucesso |
| --sb-warning | #7A5219 | Texto/ícone de atenção |
| --sb-warning-bg | #FFF5E0 | Fundo de atenção |
| --sb-error | #A13B32 | Texto/ícone de erro |
| --sb-error-hover | #842D25 | Hover da ação de descarte |
| --sb-error-bg | #FBEDEA | Fundo de erro |
| --sb-disabled | #60665E | Texto indisponível |
| --sb-disabled-bg | #E5E7E1 | Fundo indisponível |

## Tipografia

Base do navegador 16px, sem bloquear zoom. Body usa `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`; display usa `Georgia, 'Times New Roman', serif`; código usa `ui-monospace, 'SFMono-Regular', Consolas, monospace`. Não há webfont, requisição remota ou redistribuição de arquivo de fonte. Variações métricas dos fallbacks fazem parte da revisão em cada plataforma.

| Estilo | Tamanho rem (px) | Peso | Linha | Tracking |
| --- | --- | --- | --- | --- |
| Hero/editorial | clamp(2.5rem, 5.5vw, 5.5rem), 40–88px | 400 serif | 1.05 | -.035em |
| Título de tela | clamp(1.875rem, 3vw, 2.75rem), 30–44px | 400 serif | 1.12–1.18 | -.035em |
| Seção h2 | 2rem, 32px | 400 serif | 1.18 | -.02em |
| Subtítulo h3 | 1.125rem, 18px | 600 sans | 1.12 | normal |
| Corpo | 1rem, 16px | 400 sans | 1.6 | normal |
| Destaque de corpo | 1.125rem, 18px | 400 sans | 1.6 | normal |
| Label/botão/ajuda | .875rem, 14px | 600/600/400 | 1.4–1.5 | normal |
| Legenda curta/metadado | .75rem, 12px | 400 | 1.6 | normal |
| Eyebrow | .75rem, 12px | 650 sans | 1.6 | .08em, maiúsculas |

Título em sentence case; caps apenas em metadado curto. Evitar texto de corpo menor que 16px; 12/14px ficam em apoio não essencial e permanecem ampliáveis. Datas: “15 de outubro de 2027”; horas: “16h de Brasília”. Contagem em dias/horas/minutos, sem segundos a cada anúncio assistivo.

## Medidas, alinhamento e camadas

- Escala: .25/.5/.75/1/1.25/1.5/2/2.5/3/4/6rem = 4/8/12/16/20/24/32/40/48/64/96px.
- Campo/ação padrão: 3rem (48px); mínimo de alvo adotado 2.75rem (44px). Ações próximas têm pelo menos .5rem entre alvos.
- Labels a .5rem dos campos; ajuda a .5rem do campo; campos separados por 1.5rem; seções por 2–3rem.
- Container 75rem (1200px), formulário 36rem (576px), convidado 42rem (672px), e-mail 37.5rem (600px), diálogo 30rem (480px). Sempre `width:100%; max-width:...`; gutter clamp(1rem,4vw,3rem).
- Bordas 1px; estado inválido 2px com compensação de padding; radius controle .5rem (8px), card .75rem (12px), badge 999rem.
- Sem sombra em cards comuns. Popover `0 .25rem 1rem rgb(25 32 24 / 10%)`; diálogo `0 1rem 4rem rgb(25 32 24 / 20%)`.
- Camadas: base 0, sticky 10, popover 20, diálogo 30, toast 40, skip 50. `<dialog>` nativo usa top layer. Foco 3px/offset 3px; não cortá-lo com overflow do container.
- Transição 140ms em cor/borda, sem movimento decorativo; desativar com prefers-reduced-motion. Sem shimmer ou autoplay.

## Fotografia e iconografia

Fotografia é a principal fonte de cor. Preservar orientação e proporção; miniaturas/cover usam object-fit:cover e ponto focal revisado; visualização usa contain, nunca distorce. Capa proposta 16:9, miniatura 4:3; decisão final da capa está em Q02. Não sobrepor textos de navegação sobre fotos.

Ícones originais em SVG no catálogo, viewBox 24×24, traço 1.7, tamanho 20px/1.25rem; grandes de estado 32px. Usar currentColor, alinhamento central com gap 8px e aria-hidden quando ao lado de texto. Botão só com ícone precisa de nome acessível e área de 48px. Câmera, foto, vídeo, microfone, texto, envelope, cadeado, calendário, pessoas, seta, mais, aviso, play e check fazem parte do conjunto. Não usar emoji como ícone funcional.

## Contraste calculado

Meta de texto normal ≥4.5:1 e componentes/foco ≥3:1; veja [WCAG 2.2](https://www.w3.org/TR/WCAG22/). O alvo de toque de 44px é uma regra mais ampla do projeto; o mínimo AA da norma tem condições/exceções próprias. Nenhuma promessa de conformidade integral decorre da escolha de cores.

| Frente / fundo | Razão | Mínimo |
| --- | ---: | ---: |
| ink / surface | 14.93:1 | 4.5:1 |
| ink / canvas | 13.7:1 | 4.5:1 |
| muted / surface | 5.9:1 | 4.5:1 |
| muted / canvas | 5.41:1 | 4.5:1 |
| surface / primary | 14.93:1 | 4.5:1 |
| surface / primary-hover | 10.37:1 | 4.5:1 |
| surface / primary-active | 17.92:1 | 4.5:1 |
| ink / selected | 12.75:1 | 4.5:1 |
| success / success-bg | 6.23:1 | 4.5:1 |
| warning / warning-bg | 6.36:1 | 4.5:1 |
| error / error-bg | 5.77:1 | 4.5:1 |
| control-border / surface | 3.69:1 | 3:1 |
| control-border / canvas | 3.39:1 | 3:1 |
| focus / surface | 6.52:1 | 3:1 |
| focus / canvas | 5.98:1 | 3:1 |
| disabled / disabled-bg | 4.73:1 | 4.5:1 |
| surface / error | 6.59:1 | 4.5:1 |
| surface / error-hover | 8.81:1 | 4.5:1 |
| control-border / selected | 3.15:1 | 3:1 |
| focus / selected | 5.57:1 | 3:1 |

Resultados reproduzíveis em [contrast.json](contrast.json). Não usar border decorativo para identificar input. Fotografia e mídias de terceiros exigem sua própria revisão.
