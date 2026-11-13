# Identidade visual — Scrap Book

Versão 0.1 · 2026-11-13 · **Proposta aguardando revisão do responsável**.

Cartão [Create the project’s visual identity](https://trello.com/c/0YAjwM8m). Direção escolhida pelo responsável: **fotográfico minimalista, fundo neutro, poucos adornos e destaque para as mídias**. A especificação cobre o PRD completo; não equivale a implementação funcional ou aceite do produto.

## Abrir e localizar

Na raiz deste diretório: `python3 -m http.server 4174 --bind 127.0.0.1`. Abrir http://127.0.0.1:4174. O servidor serve apenas esta pasta, sem API, banco ou autenticação. Ctrl+C encerra a prévia. O catálogo requer HTTP para carregar o inventário; todos os ativos são locais, sem CDN/tracking.

| Artefato | Uso |
| --- | --- |
| [Catálogo HTML](index.html) | Visão geral, fundamentos, componentes, telas/estados e decisões. |
| [Tokens](tokens.css) / [componentes CSS](components.css) | Valores/classes reutilizáveis, independentes de framework. |
| [Fundamentos](foundations.md) | Marca, cor, tipografia, dimensões, camadas e fotografia. |
| [Componentes](components.md) | Dimensões, variações, estados e interações. |
| [Navegação e acessibilidade](navigation-accessibility.md) | Shells, fluxos, teclado, toque, responsive e semântica. |
| [Matriz de telas](screen-matrix.md) / [inventário JSON](screens.json) | Requisito, layout, estado, especificação individual e destino. |
| [Handoff HTML/CSS](handoff.md) | Snippets de composição prontos para adaptar. |
| [Decisões em aberto](decisions.md) | Questões de produto/provedor e requisitos inferidos. |
| [Ativos e prompts](assets/README.md) | Origem dos vetores/fotos, direitos e prompts completos. |
| [Contraste calculado](contrast.json) | Pares aprovados; não é auditoria integral WCAG. |

## Intenção e regras

A interface atua como moldura: papel branco ou quase branco, texto carvão, espaço entre blocos e fotografia sem filtros decorativos. O verde aparece apenas no foco/feedback semântico e nas imagens. Uma ação primária por etapa; títulos humanos, verbos claros e mensagens em português do Brasil.

A direção não presume que todos os eventos são casamentos. As fotografias de demonstração incluem celebração à mesa, aniversário e encontro no jardim; são geradas, não registros reais. Os PNGs originais são material de referência. A implementação produzirá variantes responsivas autorizadas e otimizadas; este catálogo não comprova o objetivo de carregamento 4G em três segundos.

## Fontes e compatibilização

Base: [PRD](../PRD.md), [HLD](../HLD.md), [frontend](../guidelines/frontend.md), [compatibilização](../guidelines/README.md#compatibilização-entre-prd-e-hld). Mudanças já aprovadas: confirmação de e-mail; recuperação por código Cognito de uma hora; login gerenciado; travamento por evento eventual; URLs anteriores válidas por até dois minutos. Não usar o link antigo de recuperação de trinta minutos no design novo.

Login/cadastro/recuperação são referências de marca para telas gerenciadas; a [customização Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/managed-login-brandingeditor.html) oferece configurações e ativos, mas o layout exato e a tradução precisam de validação no provedor. Este catálogo não autoriza trocar o fluxo gerenciado por autenticação própria.

## Revisão e aplicação

As especificações de telas/estados e os tokens são fonte conjunta. O catálogo demonstra a composição visual; botões/arquivos/câmera/mídias/códigos/QR usam fixtures e não executam serviços reais. O QR é explicitamente um espaço reservado. Classes/CSS podem ser levadas gradualmente ao frontend após revisão; não substituir regras, autorização ou contratos por estados visuais.

Aceite do cartão requer revisão do responsável e equipe, confirmação das questões abertas e rastreabilidade de todas as telas. A entrega técnica vai a To Deploy; **aprovação visual humana não foi registrada**. Os critérios de acessibilidade são metas de implementação, com limites das verificações relatados no ciclo de testes.

[Validação e limites](../testing/2026-11-13_12-02-01/2-execucao.md). Prévia mantida em http://127.0.0.1:4174 para revisão local.
