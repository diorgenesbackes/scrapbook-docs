# Documentação do Scrap Book

Este diretório reúne requisitos, arquitetura e padrões de desenvolvimento do Scrap Book.

## Índice e ordem de leitura

| Documento | Finalidade |
| --- | --- |
| [Guia de contribuição](AGENTS.md) | Estrutura, comandos, padrões, testes e regras de contribuição para os seis repositórios. |
| [Prompt de FDD](prompts/FDD.md) | Modelo versionado para especificações técnicas por funcionalidade. |
| [PRD](PRD.md) | Produto, regras funcionais, limites e critérios de aceitação. |
| [HLD](HLD.md) | Arquitetura aprovada, integrações, implantação, riscos e decisões pendentes. |
| [FDDs](fdd/README.md) | Especificações técnicas por funcionalidade; primeiro FDD: Gestão de Eventos. |
| [Modelagem do banco](db/README.md) | Modelo de referência, diagramas ER, dicionário físico, integridade e rastreabilidade. |
| [Projeto Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) | Scripts iniciais, changelogs, rollbacks e validação em PostgreSQL descartável. |
| [Infraestrutura local](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/main/README.md) | Docker Compose do PostgreSQL persistente e execução local do Liquibase. |
| [Guidelines](guidelines/README.md) | Regras de implementação e revisão adaptadas ao Scrap Book. |
| [Frontend](https://github.com/diorgenesbackes/scrapbook-ui/blob/main/README.md) | Projeto inicial, instalação, execução, build e testes unitários. |
| [Backend](https://github.com/diorgenesbackes/scrapbook-service/blob/main/README.md) | Solution, camadas, módulos, execução, testes unitários e contrato de saúde. |
| [Refatoração do backend](refactoring/RF-STORY-001.md) | Clean Architecture sem módulos, classes por conceito, validação e rastreabilidade. |
| [Identidade visual](visual-identity/README.md) | Proposta fotográfica minimalista, 38 telas/178 estados, tokens/componentes e catálogo HTML para revisão. |
| [Princípios](guidelines/principios.md) | Responsabilidades, Clean Architecture, SOLID e decisões. |
| [Git Flow](guidelines/git-flow.md) | Promoção seletiva, componentes, versões, publicação e recuperação. |
| [APIs síncronas](guidelines/api.md) | Contratos HTTP, erros, cursor, autorização e idempotência. |
| [Mensageria e mídias](guidelines/mensageria.md) | Outbox, consumidores, uploads, processamento e reprocessamento. |
| [Observabilidade](guidelines/observabilidade.md) | Logs, métricas, correlação, alertas e preservação da surpresa. |
| [Backend](guidelines/backend.md) | Módulos .NET, persistência, sessões, concorrência e testes. |
| [Frontend](guidelines/frontend.md) | React/TypeScript, captura, envio, visualização e acessibilidade. |
| [Liquibase](guidelines/liquibase.md) | Evolução do banco, integridade, validação e rollback. |
| [Scripts SQL](guidelines/sql.md) | Chaves inteiras, tipagem, nomes, idempotência, concorrência e reversão das alterações. |

O HLD de 2026-11-11 registra mudanças aprovadas em relação ao PRD de 2026-11-10. A [matriz de compatibilização](guidelines/README.md#compatibilização-entre-prd-e-hld) identifica essas diferenças. Os guidelines não atualizam silenciosamente o PRD nem comprovam implementação.

## Organização do workspace

Cada componente possui repositório Git independente, com branch inicial/padrão `main` e remoto `origin`. Clonar em diretórios irmãos para usar os comandos locais e helpers que dependem do workspace. Não há repositório Git agregador na raiz.

| Componente | Repositório |
| --- | --- |
| Backend .NET | [scrapbook-service](https://github.com/diorgenesbackes/scrapbook-service) |
| Documentação | [scrapbook-docs](https://github.com/diorgenesbackes/scrapbook-docs) |
| Infraestrutura | [scrapbook-infrastructure](https://github.com/diorgenesbackes/scrapbook-infrastructure) |
| Migrations PostgreSQL | [scrapbook-liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase) |
| Testes não unitários | [scrapbook-tests](https://github.com/diorgenesbackes/scrapbook-tests) |
| Frontend React | [scrapbook-ui](https://github.com/diorgenesbackes/scrapbook-ui) |

Links da base apontam para main; evidências da story usam commits ou feature/GE-STORY-001 enquanto aguardam promoção. Links internos são relativos. O guia e o prompt de FDD são versionados neste repositório. Os arquivos de mesmo nome na raiz do workspace são mantidos localmente para compatibilidade com o ambiente de trabalho.

Versionar código-fonte, documentação, configuração pública, manifests e lockfiles. Dependências instaladas, bibliotecas compiladas, arquivos gerados, credenciais, `.env`, dados do PostgreSQL e estados privados de infraestrutura ficam fora do Git, conforme o `.gitignore` de cada componente.

- `scrapbook-docs/`: requisitos, arquitetura, guidelines e modelagem em `db/`.
- `scrapbook-ui/`: projeto React/TypeScript com Vite e testes unitários.
- `scrapbook-service/`: solution .NET em camadas, gestão de eventos na feature, `/health` e unitários.
- `scrapbook-liquibase/`: base inicial de migrations, rollbacks e validações próprias.
- `scrapbook-tests/`: suítes manuais HTTP/PostgreSQL, Playwright, falhas e carga H-03.
- `scrapbook-infrastructure/`: Compose e operação do PostgreSQL local; configuração AWS manual permanece pendente. Componente acrescentado por definição do responsável em 2026-11-11.

## Organização dos próximos documentos

- `fdd/FDD-<Funcionalidade>/`: especificação `FDD-<Funcionalidade>.md` e etapas `1-planejamento.md`, `2-desenvolvimento.md`, `3-testes.md` e `4-operacao.md`, criadas conforme o trabalho avançar. A primeira especificação está em [Gestão de Eventos](fdd/FDD-Gestao-de-Eventos/FDD-Gestao-de-Eventos.md).
- `db/`: modelagem documentada em [DB-Modelagem.md](db/DB-Modelagem.md), [DB-Diagrama.md](db/DB-Diagrama.md) e [DB-Modelo-Fisico.md](db/DB-Modelo-Fisico.md), com scripts correspondentes em [scrapbook-liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md).
- `adr/`: decisões individuais, reutilizando os números ADR-001 a ADR-016 reservados no HLD.
- `testing/`: índice em `README.md` e ciclos `AAAA-MM-DD_HH-mm-ss/`, no fuso `America/Sao_Paulo`, contendo `1-plano.md` e `2-execucao.md`.

O índice de FDDs e a especificação de Gestão de Eventos já existem; suas etapas de execução e ciclos de teste estão registrados; ADRs individuais serão criados conforme o avanço. A modelagem em `db/` já está documentada. Cada FDD deve apontar para seus ciclos de teste quando existirem, sem duplicar resultados. Preservar histórico datado nas etapas e distinguir planejado, executado, aprovado, falhou e não executado.

Os repositórios de backend/frontend manterão somente testes unitários. Integração, HTTP, navegador, segurança, carga e recuperação pertencerão ao repositório de testes, com acionamento manual conforme o HLD. O Liquibase terá suas próprias validações.

Ao criar uma especificação, incluí-la neste índice. Ao mover arquivos, atualizar links relativos e preservar datas e evidências originais.

## Containers locais

O cartão [dockerizar os projetos](https://trello.com/c/NCkBjG4R) entrega Dockerfiles da API/UI e stack Compose em feature/6, dependente de GE-STORY-001/RF-STORY-001. [Operação e comandos](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/feature/6/README.md) e [evidência do ciclo](testing/2026-11-13_11-09-48/2-execucao.md). Rede interna UI→service→postgres, portas loopback e volumes preservados; identidade simulada desativada por padrão. Sem publicação AWS.
