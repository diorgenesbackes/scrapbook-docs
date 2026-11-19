# F01-STORY-002 · Backend local

A solução ASP.NET Core compõe um serviço com os projetos Api, Application, Domain, Infrastructure e UnitTests. O restore respeita os lockfiles de NuGet. A composição MVC reserva `/api/v1` para os futuros recursos de negócio. [Solução](../../../scrapbook-service/ScrapBook.sln) · [Spec da F01](../spec.md)

`/health` verifica a vitalidade do host. `/health/ready` consulta o catálogo PostgreSQL com `scrapbook_app` e verifica a presença das 24 tabelas e o acesso DML esperado. O orçamento da sondagem é configurável e começa em dois segundos. O único `ScrapBookDbContext` tem escopo por requisição. O Liquibase permanece responsável pelo DDL por acionamento explícito. [Composição](../../../scrapbook-service/ScrapBook.Api/Program.cs) · [Sondagem](../../../scrapbook-service/ScrapBook.Infrastructure/Probes/EfPostgresReadinessProbe.cs)

A API valida a configuração no início, devolve a correlação em `X-Correlation-Id` e trata falhas inesperadas com Problem Details sanitizado. A identidade de desenvolvimento vem desabilitada. O opt-in requer ambiente `Development`, um identificador de conta explícito e um token local. [Operação da API](../../../scrapbook-service/README.md)

O restore travado e o build passaram com .NET 10.0.401. O build teve zero avisos e os 22 testes xUnit passaram. [Evidência de validação](../test-funcional/2026-11-19_11-07-50/2-execucao.md)

## Ajustes da revisão técnica

A sondagem traduz falhas PostgreSQL diretas e encapsuladas pelo EF, preserva a causa interna e emite a categoria sanitizada. A identidade sintética em interfaces de rede exige o opt-in de contêiner além da habilitação em `Development`. O resumo HTTP usa eventos estruturados com rota nominal, versão, ambiente, correlação e classificação de resultado. O record `HealthResponse` passou para Domain. [Reteste](../code-review/f01-story-002.md#reteste-das-correções) · [Endpoints do Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-10.0)
