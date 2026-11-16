# Guidelines de desenvolvimento do Scrap Book

Versão inicial: **2026-11-11**.

Base: todos os nove documentos disponíveis em `Finances/expenses-docs/guidelines`, adaptados ao [PRD](../PRD.md) e ao [HLD](../HLD.md) do Scrap Book.

Estes guias orientam novas implementações e revisões. No reinício do projeto, distinguir a estrutura-alvo aprovada da implementação anterior existente: há frontend, backend com EF/PostgreSQL e funcionalidades de negócio, Liquibase, Compose e suítes manuais com evidências locais. O escopo de cada entrega está nos READMEs/FDDs/relatórios correspondentes; isso não comprova merge em main, adequação às regras novas ou deploy. A [base Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) e o [PostgreSQL local via Compose](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/main/README.md) possuem histórico próprio. Integrações reais AWS e pipelines de publicação continuam dependentes de implementação/configuração e validação identificáveis.

Atualização de arquitetura — **2026-11-13**: por solicitação do responsável, o backend passa a serviço simples em Clean Architecture, sem módulos. Classes são agrupadas por conceito e recebem seu sufixo; API usa Controllers MVC, Infrastructure usa repositórios e um ScrapBookDbContext scoped. [ADR-001](../adr/ADR-001-backend-clean-architecture.md) e [backend](backend.md) substituem as referências antigas a módulos por camada/contexto. As evidências da primeira entrega permanecem históricas. A GE já implementa Events/EF e suítes manuais; AWS, autenticação real e publicação continuam posteriores.

Revisão do backend — **2026-11-15**: regras de negócio somente nas Services de Application; repositories somente consultam/persistem dados, sem callbacks de negócio ou efeitos implícitos. Domain concentra interfaces/contratos, dados, exceptions próprias e utilitários, sem regras. Referências exatas: API → Application/Infrastructure/Domain; Application → Domain; Infrastructure → Domain; Domain → nenhum projeto. Application e Infrastructure não se conhecem. O [guia de backend](backend.md) exige código sem comentários, métodos pequenos, exceptions específicas, GlobalExceptionHandler na API e async/CancellationToken de ponta a ponta. A [revisão do ADR-001](../adr/ADR-001-backend-clean-architecture.md#revisão-vigente--2026-11-15) registra a substituição das regras anteriores; a refatoração do código é uma etapa própria.

Atualização da organização do frontend — **2026-11-15**: para o reinício do projeto, o responsável definiu as pastas globais `assets`, `components`, `config`, `context`, `features`, `hooks`, `layouts`, `lib`, `pages`, `routes`, `services`, `styles` e `utils`. O [guia de frontend](frontend.md#1-organização-de-pastas-e-responsabilidades) detalha responsabilidades, código exclusivo por página/feature, uso opcional de features conforme complexidade e direção das dependências. A alteração documental define a estrutura-alvo; a reorganização do código é uma etapa própria.

Revisão transversal aprovada — **2026-11-16**: os nove guidelines passam a contemplar manutenção de dependências e procedimentos de privacidade; falhas de Workers e configuração no startup; JSON estrito; recuperação da interface e headers de segurança; versão/checksum dos objetos, recursos de processamento e horizonte de replay; proteção da auditoria; segurança de workflows e planejamento com histórias verdes/tarefas em checklist; ownership/grants e tratamento de disputas SQL. A revisão define critérios para implementação e validação futuras, sem declarar esses controles implantados.

## Ordem de leitura

| Documento | Conteúdo |
| --- | --- |
| [Princípios comuns](principios.md) | Arquitetura, qualidade, dependências, vulnerabilidades, licenças, privacidade e decisões. |
| [Git Flow](git-flow.md) | Histórias verdes/checklists, branches, PRs, segurança dos workflows, versões, deploy e rollback. |
| [APIs síncronas](api.md) | JSON estrito, requests/responses, cursor, erros, concorrência e repetição. |
| [Mensageria e mídias](mensageria.md) | Outbox, replay/deduplicação, upload versionado, integridade e limites de processamento. |
| [Observabilidade](observabilidade.md) | Logs, métricas, tracing, integridade da auditoria, SLOs e alertas. |
| [Backend](backend.md) | Services, referências entre camadas, repositories, exceptions HTTP/Workers, startup, async/cancelamento e EF. |
| [Frontend](frontend.md) | Pastas, estado, captura/upload, recuperação de falhas, headers, acessibilidade e testes. |
| [Liquibase e PostgreSQL](liquibase.md) | Changelogs, evolução, ownership/grants, validação com identidade DML e recuperação. |
| [Scripts SQL](sql.md) | Integridade técnica, tipos/nomes, idempotência, locks/retry de transações e rollback. |

## Como interpretar as regras

- **DEVE / NÃO DEVE:** regra para novas implementações. Exceções precisam de justificativa na revisão e ADR quando alterarem arquitetura. Mudanças de produto precisam de decisão explícita.
- **RECOMENDADO:** padrão preferido, sujeito a alternativa justificada pelo contexto.
- **CONSIDERAR:** opção condicionada a uma necessidade demonstrada.
- **Definido no HLD:** decisão aprovada, ainda sujeita à implementação e validação.
- **Proposto / pendente:** detalhamento que precisa ser fechado na FDD, LLD, ADR ou configuração correspondente.
- **Implementado / validado:** somente quando houver código ou evidência identificável da execução.

Os exemplos não criam endpoints, tabelas, filas ou dependências. Nomes físicos ainda não aprovados são convenções propostas.

## Base técnica e repositórios

| Área | Base registrada | Adequação ou validação pendente |
| --- | --- | --- |
| Backend | .NET/ASP.NET Core 10, SDK 10.0.401, runtime validado 10.0.12, EF/PostgreSQL e `/health` na implementação anterior. | Adequação às responsabilidades/dependências revisadas, tratamento de erros/configuração e integrações reais. |
| Frontend | React 19.3.1, TypeScript 7.0.2, Vite 8.3.6, Oxlint 1.90.0 e Vitest 5.1.0; Node 26.13.0/npm 11.20.0, com telas/rotas existentes. | Reorganização para a estrutura-alvo, controles revisados, integrações reais e publicação Amplify. |
| Dados | Schemas físicos, mappings EF e DbContext único existentes; PKs inteiras e Liquibase exclusivo. [Modelagem](../db/README.md) e [base inicial](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) com Liquibase 4.33.0/PostgreSQL 18.6 local. | Provisionamento/patch RDS e comprovação de ownership/grants por ambiente. |
| Assíncrono | Fluxos locais e contratos documentados para Outbox, SNS/SQS, DLQ, EventBridge e MediaConvert. | Validar integrações reais, recursos, tentativas e relação entre retenção e replay por fluxo. |
| Operação | Docker/Compose locais; destinos previstos AWS us-east-1, GitHub Actions/OIDC e ECS/Amplify. | Pipelines AWS, dimensionamento, domínios, segredos, certificados e runbooks com evidência de execução. |
| Testes | xUnit, Vitest/Testing Library e suítes manuais HTTP/PostgreSQL/Playwright/carga em scrapbook-tests; Liquibase com validação própria. | Atualizar cenários para os critérios revisados e distinguir cada execução local, homologação e produção. |

Por definição posterior do responsável em 2026-11-11, a organização passou a **seis componentes**: `scrapbook-ui`, `scrapbook-service`, `scrapbook-docs`, `scrapbook-tests`, `scrapbook-liquibase` e `scrapbook-infrastructure`, todos em repositórios independentes. Infrastructure concentra a configuração operacional. Essa organização substitui a inicial de cinco componentes do HLD; os nomes físicos dos schemas e os registros históricos não recriam módulos de código.

Não transportar versões, portas, contagens de testes ou comandos do Expenses como se fossem do Scrap Book. Cada novo repositório deve documentar comandos reais e fixar dependências antes de declarar seus checks disponíveis.

## Decisões transversais

- Capacidades de negócio: **Contas e Acesso**, **Eventos e Convites**, **Registros e Mídias** e **Notificações**, sem módulos de código.
- API, produtores e consumidores na mesma aplicação e no mesmo serviço ECS, escalando juntos. Sem worker genérico separado, microsserviços ou transações distribuídas.
- Services coordenam dados/transações de cada caso de uso; mudanças e Outbox local são confirmadas juntas. Interações internas usam contratos de Domain ou mensagens conforme a consistência aprovada.
- Upload direto ao S3 privado; confirmação após persistência completa; conversão posterior não desfaz recebimento confirmado.
- Autorização por evento e por data no backend; preservar conteúdo, quantidade e remetentes até a abertura.
- Datas de negócio em Brasília, persistidas como `timestamp without time zone`; instantes técnicos e externos têm conversão explícita.
- Atualização de dados de 2026-11-11: PKs relacionais simples `id bigint GENERATED ALWAYS AS IDENTITY`, FKs compostas no nome e scripts idempotentes, com rollback obrigatório. Se a reversão não for viável, questionar a necessidade e registrar eventual exceção. Ver [SQL](sql.md).
- Sem Redis ou cache distribuído de dados de negócio. Cache de estáticos não concede acesso às lembranças.
- `dev` sem ambiente implantado; `hlg` publica em homologação; `main` publica em produção, sem aprovação adicional após merge.
- Somente unitários na automação habitual dos projetos de aplicação; demais suítes por acionamento manual. A obrigatoriedade dos cenários do PRD permanece.

## Compatibilização entre PRD e HLD

O HLD registra decisões aprovadas posteriores ao PRD. Aplicar os ajustes explícitos abaixo e rastreá-los na futura atualização do PRD/FDD; isso não estabelece precedência irrestrita para qualquer divergência nova.

| Tema | PRD original | Decisão registrada no HLD para os guidelines |
| --- | --- | --- |
| Cadastro | Sem confirmação inicial de e-mail. | Confirmação no cadastro e do novo e-mail antes da substituição. |
| Recuperação | Link SES de 30 minutos e uso único. | Código Cognito válido por uma hora; integrar revogação das sessões locais. |
| Primeiro registro | Travamento imediato de dados e convites. | Evento via Outbox e consumo; meta de até 5 segundos em condições normais, com janela eventual aceita. |
| Mudança para bloqueio | Impedir acesso inclusive por URLs anteriores. | Impedir novas URLs; URLs existentes têm validade de até 2 minutos. Conteúdo entregue e transferências iniciadas exigem tratamento explícito. |
| Abertura e e-mail | Liberação e aviso após a abertura. | Autorização por data em cada acesso; Scheduler diário às 00h de Brasília apenas dispara notificações. |

Política de senha, detalhes OIDC/CSRF, revogação e capacidade das telas gerenciadas devem ser comprovados no Cognito. Não substituir uma pendência por uma afirmação de suporte.

## Fluxo de uma mudança

1. Vincular requisito, história entregável no Trello com etiqueta verde, capacidade de negócio e repositórios afetados; detalhar tarefas somente em checklist, conforme o [Git Flow](git-flow.md#histórias-entregáveis-e-tarefas-em-checklist).
2. Definir contrato, permissões, limites, estados, prazos e fonte de verdade.
3. Identificar consistência, transação local, idempotência, mensagens e efeitos externos.
4. Implementar a menor entrega completa, com SQL, testes e recuperação pertinentes.
5. Executar checks automáticos e acionar os testes manuais necessários; registrar revisão, ambiente e resultados.
6. Atualizar FDD, modelo e ADR quando aplicável; promover conforme o [Git Flow](git-flow.md).

Não exigir testes novos para correções apenas textuais. Para autorização temporal, persistência, sessões, filas e integridade de conjuntos, cobrir limites e falhas observáveis.

## Adoção e manutenção

- Manter versões e comandos reais atualizados nos repositórios existentes e registrar suas evidências de execução.
- Detalhar contratos HTTP, mensagens, estados e modelo físico junto de cada funcionalidade.
- Formalizar as ADRs numeradas no HLD sem reutilizar seus números para decisões diferentes.
- Configurar checks, pipelines, telemetria e permissões; validar seu funcionamento antes de chamá-los de implementados.
- Medir carga, privacidade, acessibilidade e recuperação antes do lançamento. Metas de disponibilidade, RTO e RPO não são evidência de atendimento.

Atualizar os guias quando mudar arquitetura, contrato transversal ou decisão operacional. Preservar evidências históricas conforme a [organização da documentação](../README.md).
