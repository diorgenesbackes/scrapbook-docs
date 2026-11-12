# Repository Guidelines

## Project Structure & Module Organization

Read [index](README.md), [PRD](PRD.md), [HLD](HLD.md), and [guidelines](guidelines/README.md). Frontend: `scrapbook-ui/src/{app,features,shared,test}`. Backend: `ScrapBook.sln` and five project directories, including UnitTests, directly under `scrapbook-service/`. Module folders: Identity, Events, Records, Notifications.

Other components: `scrapbook-docs`, `scrapbook-tests`, `scrapbook-liquibase`, and `scrapbook-infrastructure`. Each repository uses `main`.

## Build, Test, and Development Commands

Inside `scrapbook-ui/` (Node 26.13.0, npm 11.20.0):

- `npm ci`: install locked dependencies.
- `npm run dev`: start Vite at `http://127.0.0.1:5173`.
- `npm run check`: lint, type checking, build, and unit tests.
- `npm test`: unit tests once.

Inside `scrapbook-service/` (SDK 10.0.401):

- `dotnet restore ScrapBook.sln --locked-mode`: restore pinned dependencies.
- `dotnet build ScrapBook.sln --no-restore`: build all projects.
- `dotnet test ScrapBook.sln --no-build --no-restore`: run unit tests.
- `dotnet run --project ScrapBook.Api --launch-profile http`: API on port 5080.

## Coding Style & Naming Conventions

Use Markdown headings, relative links, and Portuguese documentation. Keep guideline filenames lowercase, including `git-flow.md`. Preserve PRD/HLD filenames.

Use English identifiers, strict TypeScript, single-quoted strings, and two-space TypeScript/YAML indentation. Oxlint checks hooks, accessibility, and shared imports. C# uses four spaces, PascalCase types/methods, camelCase locals, and `Async` suffixes.

## Architecture & Documentation Decisions

Preserve the modular .NET monolith: Accounts/Access, Events/Invitations, Records/Media, and Notifications. API and consumers share one application; module-local transactions include Outbox messages. Liquibase exclusively manages schema changes.

Follow [SQL conventions](guidelines/sql.md): integer identity PKs, safe replay, and rollback. Question changes without viable reversal.

Follow the PRD/HLD compatibility matrix. Distinguish proposed, implemented, and validated behavior; preserve historical evidence. Business timestamps represent `America/Sao_Paulo`.

## Testing Guidelines

Keep unit tests in backend/frontend; run non-unit suites manually from scrapbook-tests. In scrapbook-liquibase, `python3 tests/verify.py` validates migrations in disposable Docker PostgreSQL.

Frontend: Vitest/React Testing Library with `*.test.ts(x)`; backend: xUnit with `*Tests` classes. Playwright/k6 remain planned. No coverage threshold exists. Prioritize authorization, isolation, upload atomicity, retries, duplicates, and recovery. Report skipped tests.

## Commit & Pull Request Guidelines

Use exclusively the [Scrap Book Trello board](https://trello.com/b/6QX5TIHS/scrap-book). Link tasks, dependencies, acceptance criteria, and execution evidence to their story.

Use Conventional Commits, e.g. `docs(guidelines): clarify upload confirmation`. Commit manifests and lockfiles; exclude installed libraries, generated artifacts, and secrets.

Follow [Git Flow](guidelines/git-flow.md): features originate from main and promote separately to dev, hlg, then main. PRs link Trello and document behavior, dependencies, validation, migrations, and recovery.

## Security & Configuration

Keep credentials, tokens, usable signed URLs, and real guest content out of commits and examples. Authorize event access and opening dates server-side; protect blocked content, sender identities, and counts.
