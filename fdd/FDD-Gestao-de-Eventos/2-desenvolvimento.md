# Desenvolvimento — Gestão de Eventos

Início: 2026-11-12. Story: [GE-STORY-001](https://trello.com/c/kIAhFOno).

## Fluxo solicitado

Cada leva vai para **Todo** antes da implementação. Cada tarefa iniciada vai para **Doing**. Ao iniciar a validação funcional, de carga/performance pertinente à leva, as tarefas vão para **Test**. Somente tarefas aprovadas seguem para **To Deploy**. Não mover automaticamente para Done nem executar deploy nesta etapa.

O estado atual permanece no quadro [Scrap Book](https://trello.com/b/6QX5TIHS/scrap-book). Evidências e commits serão acrescentados conforme a execução real; preparação não comprova critérios funcionais da story.

## Leva 1 — preparação local

Selecionadas: GE-001…GE-008. As oito tarefas foram movidas para Todo antes do desenvolvimento. A story foi movida para Doing. Branch de trabalho: `feature/GE-STORY-001`, originada de `main` nos componentes alterados.

### GE-001 — dependências

Selecionados no catálogo oficial NuGet: EF Core/Relational **10.0.15**, provider PostgreSQL **10.0.3** e Npgsql **10.0.3**. Referências diretas e lockfiles são versionados; os pacotes instalados ficam fora do Git. Não há migrations EF.

Fontes: [EF Core 10](https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/whatsnew), [provider Npgsql 10](https://www.npgsql.org/efcore/release-notes/10.0.html), [catálogo EF](https://api.nuget.org/v3-flatcontainer/microsoft.entityframeworkcore/index.json) e [catálogo Npgsql](https://api.nuget.org/v3-flatcontainer/npgsql.entityframeworkcore.postgresql/index.json).

O PostgreSQL local foi iniciado pelo Compose existente, preservando o volume. GitHub por SSH permanece disponível para envio das branches. A integração de PRs foi sugerida e ainda depende da conexão do GitHub; isso não bloqueia a implementação local. Nenhuma promoção a hlg/main foi realizada.

### Validação da leva

GE-001…GE-008 passaram para Test antes das validações manuais e foram aprovadas no escopo da preparação. Evidência: [ciclo 2026-11-12_20-04-48](../../testing/2026-11-12_20-04-48/2-execucao.md). A baseline de /health não comprova carga/performance de Events; essa avaliação permanece na GE-107.

Commits: service `b86e996`/`64c5739`, infrastructure `061c408`, tests `245d50d`, UI `ea62ae4`. Chaves locais e credenciais não foram versionadas. O ID da fixture é sempre o valor retornado/persistido pelo bootstrap.
