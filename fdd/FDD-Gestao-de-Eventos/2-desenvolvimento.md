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

## Leva 2 — persistência e domínio

Selecionadas GE-009…GE-034; GE-047 incluída por compartilhar a projeção de capacidades com GE-018. Os cartões foram selecionados em Todo e iniciados em Doing antes da implementação de seus grupos. O filtro interno por vínculo começou junto do contexto; GE-017 foi atualizado para Doing na consolidação desse grupo, sem antecipar uma data de início fictícia.

Implementados contextos separados, mapeamentos explícitos, grants existentes sem DDL, porta de Identity, identidade opt-in/loopback, CORS/origem, catálogo SQLSTATE, unidade de trabalho sem retry e regras de domínio. Snapshot de Events não lê os detalhes do primeiro registro. A projeção de capacidades usa uma única amostra temporal para estado e serverTime.

Passagem para Test em 2026-11-12, conforme histórico dos cartões. Evidência: [ciclo de persistência/domínio](../../testing/2026-11-12_20-36-51/2-execucao.md). Service `3733180`; probes `5b95c7e`. A falha inicial de permissão no campo de capa foi corrigida e retestada, sem relaxar grants. Não há publicação em main/hlg.

As 27 tarefas da leva 2 (GE-009…GE-034 + GE-047) foram confirmadas em To Deploy no quadro. A integração Trello retornou Transport closed durante a atualização; a movimentação foi concluída/conferida no navegador. Após a confirmação do responsável, a integração voltou a responder e o quadro foi reconciliado.

## Leva 3 — casos de uso e HTTP

Selecionadas GE-035…GE-046 e GE-048…GE-059 em Todo antes do desenvolvimento. Plano de testes: [CRUD HTTP](../../testing/2026-11-12_21-45-00/1-plano.md). A GE-047 já foi atendida na leva 2.

Implementados store/casos de uso, auditoria atômica, lock e versão, cursor protegido, cinco endpoints, JSON estrito, budget/correlação e OpenAPI. GE-086 foi incluída antes de alterar diagnóstico do EF. A suíte manual utiliza PostgreSQL exclusivo e 47 migrations inalteradas; ver [execução HTTP](../../testing/2026-11-12_21-45-00/2-execucao.md).

## Leva 4 — interface, instrumentação e testes finais

GE-060…GE-082 foram selecionadas em Todo antes de implementar a UI; GE-083…GE-085 seguiram o mesmo fluxo. GE-087…GE-108 implementam/validam laboratório, HTTP, navegador, falhas, privacidade e performance. A preparação do laboratório precedeu a execução das suítes. A primeira inicialização TCP aguardou o servidor definitivo após correção de readiness.

Cartões retornaram a Doing quando houve falha/correção: 415 antes do handler, erro de conexão encapsulado pelo EF, atualizações de formulário por snapshot e foco/ref. Não houve retrodatação de transições. Os detalhes e retestes estão no [ciclo integrado](../../testing/2026-11-12_22-23-17/2-execucao.md).

Rotas usam React Router declarativo. IDs/versões permanecem strings; calendário civil não usa timezone do aparelho. Dois formulários salvam independentemente; conflito e resposta perdida preservam rascunho e exigem nova decisão. Logs JSON técnicos e ActivitySource/Meter locais não implementam coletor AWS nem expõem dados de recebimento.

77 unitários backend, 39 frontend, 14 jornadas finais, HTTP/PostgreSQL e H-03 aprovados localmente. A revisão final acrescentou 500 ao OpenAPI e conferiu HTTP de estrutura travada e edição independente de acesso; o dataplano medido em H-03 permaneceu inalterado. A massa de teste foi descartada sem tocar no volume do usuário.

## Consolidação

GE-109…GE-112 documentam comandos, recuperação sem apagar dados, FDD 0.2, matriz dos 26 aceites e relatório por task. Todos os cartões terminam em To Deploy após validação no escopo descrito. Nenhum merge de ambiente, deploy, PR ou aceite humano é anunciado. Os commits são agrupados por componente na feature; o relatório identifica revisões entregáveis, sem afirmar um commit exclusivo para cada task.

Consultar [matriz](3-testes.md), [operação](4-operacao.md) e [relatório](5-relatorio.md). Horários são transições reais do Trello; não representam horas ativas ou esforço exclusivo. A falha temporária da integração e o uso restrito do navegador estão registrados na leva 2.
