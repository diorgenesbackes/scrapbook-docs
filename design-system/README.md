# Scrap Book — Sistema de design

Proposta para revisão derivada da identidade visual fotográfica minimalista. As telas e mensagens estão em português do Brasil.

## Abrir

Execute na pasta do sistema de design:

```sh
python3 -m http.server 4174 --bind 127.0.0.1
```

Abra [o catálogo local](http://127.0.0.1:4174). O catálogo usa HTML, CSS, JavaScript e ativos locais. Encerre o servidor com Ctrl+C.

## Consultar

- [Fundamentos](foundations.md) — marca, cores, tipografia, espaçamento e fotografia.
- [Tokens CSS](tokens.css) e [componentes CSS](components.css) — estilos reutilizáveis.
- [Componentes](components.md) — variantes, dimensões e comportamento.
- [Matriz de telas](screen-matrix.md) e [inventário JSON](screens.json) — 38 superfícies, 178 estados e 22 requisitos funcionais. Cada superfície tem uma especificação individual.
- [Navegação e acessibilidade](navigation-accessibility.md) — teclado, toque, layout responsivo e semântica.
- [Aplicação e verificação](implementation.md) — estados críticos, regras do produto e critérios WCAG.
- [Handoff](handoff.md), [decisões abertas](decisions.md), [ativos](assets/README.md) e [pares de contraste](contrast.json).

Fontes entregues pelo solicitante: visual-identity.zip, PRD.md e HLD.md. O material original é mantido nos repositórios de origem. Esta pasta pode ser distribuída como referência independente; aprovação visual, validação no Cognito e conformidade integral dependem de revisão do produto integrado.
