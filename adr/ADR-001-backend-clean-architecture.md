# ADR-001 — Backend simples em Clean Architecture

Data: 2026-11-13. Estado: decisão explícita do responsável; implementação na RF-STORY-001. Substitui a reserva original de ADR-001 para monólito modular no HLD 1.0.

## Contexto e decisão

O responsável solicitou desmodularizar scrapbook-service, manter Clean Architecture como um serviço simples, usar Controllers na API e agrupar classes por conceito, com o respectivo sufixo no nome.

Manter os projetos Domain, Application, Infrastructure, API e UnitTests diretamente na raiz da solution. Remover Modules/Identity/Events/Records/Notifications e Endpoints. Não recriar essa divisão dentro de Services/Repositories ou em namespaces. API contém EventController e HealthController MVC; os contratos usam Requests/Responses. Application contém interfaces, serviços, comandos e modelos. Domain contém entidades, políticas, exceções e relógio. Infrastructure contém repositórios, entidades de persistência, configurações e um ScrapBookDbContext scoped.

Uma classe/record por arquivo com nome correspondente; sufixos Controller, Service, Repository, Entity, Model, Command, Request, Response, Mapper, Validator, Middleware, Configuration, Options, Policy, Clock, Scope, Reader, Writer, Formatter, Transformer, Extensions e Exception identificam o conceito. Interfaces usam I mais o conceito, enums usam Enum, testes usam Tests e doubles usam Stub. Program.cs mantém o papel de entrada/composição.

## Consequências

EventController usa IEventService, e EventService depende de IEventRepository/IUnitOfWork. Repositórios compartilham o contexto da requisição; mapeamentos EF ficam em classes IEntityTypeConfiguration próprias. Mantêm-se o modelo imutável do domínio e entidades técnicas de persistência para respeitar campos gerados e grants restritos do banco.

MVC usa requests tipados e um input formatter que reutiliza a leitura JSON estrita; a fronteira de erro continua devolvendo Problem Details e orçamento/correlação existentes. HealthController conserva a resposta mínima de saúde, independente de ator/banco.

Os schemas físicos, contratos HTTP, propósitos criptográficos do cursor e nomes de configuração/telemetria existentes permanecem compatíveis. São contratos de armazenamento/operação, sem definir módulos de código. Não há nova migration, divisão de implantação, banco adicional ou mudança funcional autorizada por esta refatoração.

A RF depende da GE-STORY-001 ainda não promovida: feature/RF-STORY-001 parte do conteúdo entregue dessa história e registra a dependência para futura integração seletiva. Evidências anteriores da GE permanecem históricas; a estrutura vigente é descrita no HLD 1.1, guidelines e relatório RF.

## Validação

Build/unitários e regressão HTTP/PostgreSQL, navegador, privacidade e H-03 no laboratório descartável, com resultados em [RF-STORY-001](../refactoring/RF-STORY-001.md). Os serviços do usuário permanecem parados ao finalizar.
