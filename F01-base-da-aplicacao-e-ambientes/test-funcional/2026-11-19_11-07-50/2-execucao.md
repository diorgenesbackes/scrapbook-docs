# Execução da validação local da F01-STORY-002

Em 19/11/2026, os cenários rodaram em macOS 26.6.2 ARM64, Docker 29.8.2 com Compose 5.6.0, PostgreSQL 18.6 e Liquibase 4.33.0. O teste usou alterações locais de `feature/F01-STORY-002` a partir de `dev`. Bases da execução em `dev` por componente: serviço `9a32dce`, UI `1ecfe5f`, infraestrutura `f2b4388`, testes `175565b`, documentação `b45b319` e Liquibase `1adeced`. [Plano](1-plano.md)

## Resultados

| Critério | Evidência observada | Resultado |
| --- | --- | --- |
| CA1 · Execução local | O banco vazio permaneceu sem o schema de aplicação após o início da API. O preparo explícito com Liquibase fez a prontidão passar de `503` a `200`. As publicações em loopback, o proxy, os cabeçalhos e a execução de API e UI sem root passaram. | Aprovado |
| CA2 · Recuperação | A API permaneceu `200` em `/health` com PostgreSQL parado. `/health/ready` respondeu `503` e recuperou `200` após o retorno da dependência. A conta sintética e o arquivo de chaves persistiram após recriação. | Aprovado |
| CA3 · Configuração | O host recusou a opção de conexão inválida e a habilitação da identidade local em `Production`. Os logs identificaram a opção e omitiram os segredos de teste. Os unitários confirmaram a identidade desabilitada por padrão. | Aprovado |
| CA4 · Interface | O Playwright verificou início, fallback, reflow, foco, tecla de atalho e retentativa nos cinco perfis abaixo. O axe não detectou violações nas regras habilitadas na página inicial. Faltam ensaios manuais de teclado, contraste e leitor de tela na matriz de navegadores e dispositivos físicos. | Parcial |

| Componente | Comando e resultado |
| --- | --- |
| Serviço | `dotnet restore ScrapBook.sln --locked-mode`, `dotnet build ScrapBook.sln --no-restore` e `dotnet test ScrapBook.sln --no-build --no-restore` passaram com SDK 10.0.401. Build sem avisos e 22 testes xUnit aprovados. |
| UI | `npm ci` e `npm run check` passaram com Node 26.13.0 e npm 11.20.0. Lint, build e sete testes Vitest aprovados. |
| Stack e navegador | `python3 docker_stack.py --browser` retornou `0`. O cenário Compose e os dez casos Playwright passaram. O runner também foi exercitado com porta e senha sintéticas conflitantes no ambiente para verificar o isolamento da execução. |

## Matriz automatizada

| Perfil | Motor e versão | Alcance | Resultado |
| --- | --- | --- | --- |
| Chromium desktop | Chromium 158.0.8127.18 | macOS, viewport de 320 px | 2 de 2 |
| Firefox desktop | Firefox 159.0 | macOS | 2 de 2 |
| WebKit desktop | WebKit 27.2 | macOS | 2 de 2 |
| Android emulado | Chromium 158.0.8127.18 | Perfil Pixel 7 | 2 de 2 |
| iPhone emulado | WebKit 27.2 | Perfil iPhone 13 | 2 de 2 |

A primeira execução obteve oito aprovações. Nos dois perfis WebKit, Tab simples não focalizou o atalho. O teste passou a usar Option+Tab para alcançar links segundo a configuração padrão documentada do Safari no Mac. No reteste, Enter levou o foco ao conteúdo nos dois perfis. [Atalhos do Safari](https://support.apple.com/guide/safari/keyboard-shortcuts-and-gestures-cpsh003/mac)

A matriz das duas versões principais de Chrome, Safari, Edge e Firefox nas plataformas aplicáveis, incluindo Android e iPhone físicos, permanece pendente. A emulação e o axe documentam parte da acessibilidade. O aceite integral da CA4 e o item T11 dependem dos testes manuais, incluindo anúncio dos estados com leitores de tela. [Runner Compose](../../../../scrapbook-tests/docker_stack.py) · [Casos de navegador](../../../../scrapbook-tests/playwright/foundation.spec.ts)
