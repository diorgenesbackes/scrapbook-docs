# Revisão técnica da F05-STORY-001

Escopo comparado com `dev` em `d5f940e` no serviço, `b56a764` na interface e `822778b` nos testes. A revisão cobriu o GET anônimo, a projeção PostgreSQL, a janela calculada pela Service, a troca de contexto no navegador, os headers de hospedagem e a suíte descartável. [Spec](../spec.md) · [Card](https://trello.com/c/bRjMjxdL)

| Achado | Correção e evidência |
| --- | --- |
| Carregamento sob demanda atrasava o descarte de consulta ao trocar um evento autenticado. | A separação entre rotas de convidado e as demais foi movida para `App`. Os testes de troca de contexto dos dois fluxos passam. |
| O `Suspense` mantinha o estado de carregamento das rotas dos responsáveis com o relógio de navegador pausado. | A importação sob demanda é resolvida antes de renderizar o componente de rotas. A página pública mantém o bundle separado. Um ciclo de revisão da correção conferiu desmontagem, falha de importação e reentrada. A regressão passou nos cinco projetos sem ajustes no teste. |
| `CA1822` indicou que o cálculo da janela era independente de instância. | A Service mantém o cálculo estático e reutilizável. A segunda verificação de estilo passou. |
| O padrão do pool de conexões esgotava slots no pico da consulta pública. | A configuração usa limite padrão de 32 por processo e preserva máximo explícito e mínimo configurado. A revisão da correção validou as opções, a tradução do erro de slots e o reteste. [Perfil e execução](../test-performance/f05-story-001.md) |

| Check | Resultado do candidato |
| --- | --- |
| .NET SDK 10.0.401, restore bloqueado pelo lockfile, build sem incremento e `dotnet format --verify-no-changes --no-restore --severity info` | Sucesso. Compilação final com zero warnings e zero erros. |
| xUnit | 107 testes passaram. |
| Node 26.13.0, npm 11.20.0, Oxlint, TypeScript, Vite e Vitest | `npm run check` passou com 26 unitários. Lint e build finais sem warnings. |
| npm ci pelo lockfile | O npm informou que o script de instalação opcional de `fsevents@2.3.3` fica fora de `allowScripts`. Os checks passaram com a configuração vigente. |
| HTTP e PostgreSQL local | Sucesso na stack descartável com o Liquibase vigente. A busca usa a projeção pública e filtra por igualdade no campo com índice único. |
| Artefato do Amplify | Geração local confirmou headers públicos, headers gerais e `Cache-Control` do manifesto de release. |

Revisão final sem achados no escopo técnico. O resultado integrado e as verificações restantes constam nos relatórios da feature.
