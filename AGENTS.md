# Repository Guidelines

## Project Structure

Read [index](README.md), [PRD](PRD.md), [HLD](HLD.md), and [guidelines](guidelines/README.md). The workspace has six independent repositories: service, UI, docs, tests, Liquibase and infrastructure. Their default branch is main.

ScrapBook.sln and five project directories sit directly under scrapbook-service, without src/test wrappers. Frontend uses scrapbook-ui/src/{app,features,shared,test}.

## Architecture and Naming

Use one backend service with Clean Architecture. Domain holds pure entities, policies, models, requests, responses, builders, enums, mappers and readers using only .NET base libraries. Application exclusively owns Services and defines ports/commands. Neither references ASP.NET or EF. Infrastructure implements persistence/integrations; API contains controllers and technical HTTP adapters.

Group classes by concept, without Modules or functional subfolders. Extract pure behavior into Domain; keep framework adapters in API/Infrastructure with explicit names: HttpJsonBodyReader, EventMvcJsonInputFormatter, HealthHttpResponseWriter, EfPostgresExceptionMapper. POCO PersistenceEntity types live in Domain; EF mappings and the single scoped ScrapBookDbContext stay in Infrastructure. Never move framework dependencies into Domain.

Every declared type has its concept suffix: EventController, EventService, EventRepository, EventEntity, CreateEventRequest, EventResponse, EventPolicy, EventApplicationException. Keep one type per matching filename. Interfaces retain I plus their service/repository concept; tests end in Tests. Program.cs is the composition entry point.

C# uses four spaces, PascalCase types/methods, camelCase locals and Async methods. TypeScript uses strict mode, two spaces and single quotes. Document in Portuguese with relative links; guideline filenames stay lowercase.

## Build and Tests

Backend, SDK 10.0.401:

- dotnet restore ScrapBook.sln --locked-mode
- dotnet build ScrapBook.sln --no-restore
- dotnet test ScrapBook.sln --no-build --no-restore

Frontend, Node 26.13.0/npm 11.20.0: npm ci, npm run dev and npm run check.

Use xUnit and Vitest/Testing Library for unit tests. Keep HTTP/PostgreSQL/Playwright/load tests manual in scrapbook-tests. Liquibase owns migrations and its disposable validation suite. Report failures and skipped scenarios; no coverage threshold exists.

## Contribution and Security

Track work exclusively in [Scrap Book](https://trello.com/b/6QX5TIHS/scrap-book): Todo → Doing → Test → To Deploy. Use Conventional Commits and [Git Flow](guidelines/git-flow.md); identify dependent features and their base. Preserve historical evidence and distinguish local validation from deployment.

Commit manifests/lockfiles; exclude libraries, builds, credentials and private state. Follow [SQL](guidelines/sql.md), preserve business dates in America/Sao_Paulo, and authorize event access server-side. Development identity is explicit, local and disabled by default.
