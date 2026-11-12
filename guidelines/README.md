# Guidelines de desenvolvimento do Scrap Book

Versão inicial: **2026-11-11**.

Base: todos os nove documentos disponíveis em `Finances/expenses-docs/guidelines`, adaptados ao [PRD](../PRD.md) e ao [HLD](../HLD.md) do Scrap Book.

Estes guias orientam novas implementações e revisões. O workspace contém a base do frontend, a solution do backend e a consulta operacional `/health` entre eles. A [base Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) foi criada em 2026-11-12, com validação local e rollbacks, e aplicada ao [PostgreSQL local via Compose](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/main/README.md). Funcionalidades de negócio, persistência EF, integrações, infraestrutura AWS e pipelines ainda serão implementados. Os exemplos dessas funcionalidades permanecem ilustrativos.

## Ordem de leitura

| Documento | Conteúdo |
| --- | --- |
| [Princípios comuns](principios.md) | Clean Architecture, Clean Code, SOLID, padrões, revisão e ADRs. |
| [Git Flow](git-flow.md) | Branches, promoção seletiva, PRs, versões independentes, deploy e rollback. |
| [APIs síncronas](api.md) | Requests, responses, paginação por cursor, erros, concorrência e repetição. |
| [Mensageria e mídias](mensageria.md) | Complemento do Scrap Book para Outbox, SQS/SNS, upload e preparação das mídias. |
| [Observabilidade](observabilidade.md) | Logs, métricas, tracing, auditoria, SLOs e alertas. |
| [Backend](backend.md) | C#, módulos, DI, EF Core, autorização, transações e testes. |
| [Frontend](frontend.md) | React/TypeScript, estado, captura, upload, galeria, acessibilidade e testes. |
| [Liquibase e PostgreSQL](liquibase.md) | Changelogs, SQL, integridade, evolução, entrega e recuperação. |
| [Scripts SQL](sql.md) | PK inteira, tipos e tamanhos, nomenclatura, idempotência, concorrência e rollback obrigatório. |

## Como interpretar as regras

- **DEVE / NÃO DEVE:** regra para novas implementações. Exceções precisam de justificativa na revisão e ADR quando alterarem arquitetura. Mudanças de produto precisam de decisão explícita.
- **RECOMENDADO:** padrão preferido, sujeito a alternativa justificada pelo contexto.
- **CONSIDERAR:** opção condicionada a uma necessidade demonstrada.
- **Definido no HLD:** decisão aprovada, ainda sujeita à implementação e validação.
- **Proposto / pendente:** detalhamento que precisa ser fechado na FDD, LLD, ADR ou configuração correspondente.
- **Implementado / validado:** somente quando houver código ou evidência identificável da execução.

Os exemplos não criam endpoints, tabelas, filas ou dependências. Nomes físicos ainda não aprovados são convenções propostas.

## Base técnica e repositórios

| Área | Decisão vigente | A definir antes da implementação |
| --- | --- | --- |
| Backend | .NET/ASP.NET Core 10, SDK 10.0.401 e runtime validado 10.0.12; solution em camadas e `/health`. | EF Core, provider PostgreSQL, persistência e integrações. |
| Frontend | React 19.3.1, TypeScript 7.0.2, Vite 8.3.6, Oxlint 1.90.0 e Vitest 5.1.0; Node 26.13.0/npm 11.20.0. Publicação estática prevista no Amplify. | Roteamento, funcionalidades, integração real e configuração de publicação. |
| Dados | RDS PostgreSQL, schemas e `DbContext` por módulo; PKs inteiras e Liquibase exclusivo. [Modelagem](../db/README.md) e [base inicial](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) com Liquibase 4.33.0/PostgreSQL 18.6 local. | Provisionamento/patch RDS, identidade DDL/DML e mappings EF. |
| Assíncrono | Outbox, SNS, SQS Standard, DLQ, EventBridge e MediaConvert. | Contratos detalhados, concorrência, tentativas, retenção e perfis. |
| Operação | AWS `us-east-1`, GitHub Actions/OIDC, ECS Fargate/ECR e configuração manual da infraestrutura. | Dimensionamento, domínios, segredos, certificados e runbooks. |
| Testes | Backend com xUnit 2.9.3; frontend com Vitest, React Testing Library e jsdom. Demais testes manuais em projeto separado; Playwright e k6 previstos. | Inventário, suítes não unitárias e dados de teste. |

Por definição posterior do responsável em 2026-11-11, a organização passou a **seis componentes**: `scrapbook-ui`, `scrapbook-service`, `scrapbook-docs`, `scrapbook-tests`, `scrapbook-liquibase` e `scrapbook-infrastructure`. O último concentra a futura configuração operacional. Todos os diretórios existem; frontend e backend possuem projetos iniciais e os demais componentes técnicos aguardam implementação. A criação de repositórios Git e remotos continua separada. Essa atualização substitui a organização inicial de cinco repositórios do HLD.

Não transportar versões, portas, contagens de testes ou comandos do Expenses como se fossem do Scrap Book. Cada novo repositório deve documentar comandos reais e fixar dependências antes de declarar seus checks disponíveis.

## Decisões transversais

- Módulos: **Contas e Acesso**, **Eventos e Convites**, **Registros e Mídias** e **Notificações**.
- API, produtores e consumidores na mesma aplicação e no mesmo serviço ECS, escalando juntos. Sem worker genérico separado, microsserviços ou transações distribuídas.
- Cada módulo é dono de seus dados e transações; mudanças e Outbox local são confirmadas juntas. Interações internas usam contratos ou mensagens conforme a consistência aprovada.
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

1. Vincular requisito, história no Trello, módulo e repositórios afetados.
2. Definir contrato, permissões, limites, estados, prazos e fonte de verdade.
3. Identificar consistência, transação local, idempotência, mensagens e efeitos externos.
4. Implementar a menor entrega completa, com SQL, testes e recuperação pertinentes.
5. Executar checks automáticos e acionar os testes manuais necessários; registrar revisão, ambiente e resultados.
6. Atualizar FDD, modelo e ADR quando aplicável; promover conforme o [Git Flow](git-flow.md).

Não exigir testes novos para correções apenas textuais. Para autorização temporal, persistência, sessões, filas e integridade de conjuntos, cobrir limites e falhas observáveis.

## Adoção e manutenção

- Fixar versões e comandos nos repositórios quando forem criados.
- Detalhar contratos HTTP, mensagens, estados e modelo físico junto de cada funcionalidade.
- Formalizar as ADRs numeradas no HLD sem reutilizar seus números para decisões diferentes.
- Configurar checks, pipelines, telemetria e permissões; validar seu funcionamento antes de chamá-los de implementados.
- Medir carga, privacidade, acessibilidade e recuperação antes do lançamento. Metas de disponibilidade, RTO e RPO não são evidência de atendimento.

Atualizar os guias quando mudar arquitetura, contrato transversal ou decisão operacional. Preservar evidências históricas conforme a [organização da documentação](../README.md).
