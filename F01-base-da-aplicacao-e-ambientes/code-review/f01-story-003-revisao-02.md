# F01-STORY-003 · Segunda revisão técnica em 20/11/2026

## P1 · A expressão do caminho do ZIP invalida o workflow da UI

A correção colocou `SITE_ARCHIVE: ${{ runner.temp }}/amplify-site/site.zip` em `jobs.publish.env`. O GitHub Actions restringe os contextos nesse campo, e `runner` só está disponível nos campos de execução apropriados, como `steps.env` e `steps.with`. A verificação local de contextos identificou a expressão na linha 89 e saiu com código 1. Isso impede o workflow de iniciar. Inicialize `SITE_ARCHIVE` em um passo do job, a partir de `RUNNER_TEMP`, e exporte-o para a conferência e a publicação. A expressão em `steps.with.path` usa um contexto permitido.

A linha foi introduzida pelo diff da correção que separou os jobs de build e publicação. O teste local dos scripts injeta o caminho diretamente e, por isso, não verifica a resolução da expressão de workflow. [Workflow da UI](../../../scrapbook-ui/.github/workflows/publish-hlg.yml#L89) · [Contextos do GitHub Actions](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#context-availability) · [Schema do runner](https://github.com/actions/runner/blob/main/src/Sdk/DTPipelines/workflow-v1.0.json#L217)

## Verificação dos achados anteriores

| Ponto | Resultado no diff revisado |
| --- | --- |
| Bootstrap com administrador sem SUPERUSER | A suíte preparou os bancos descartáveis com `NOSUPERUSER` e `CREATEROLE`. Uma sondagem independente confirmou que uma falha depois do grant temporário preserva `INHERIT TRUE, SET FALSE` da associação preexistente. |
| Reexecução parcial | Os três workflows recebem a chave da reserva e a conferem antes de assumir credenciais AWS. A API e a UI recusaram a chave de outra tentativa em simulação. |
| Timeout da migração | O workflow cria o registro pendente antes de `RunTask`. Os testes confirmaram o bloqueio enquanto a tarefa estiver em andamento e a reconciliação quando a tarefa está concluída com digest e código de saída válidos. |
| Proveniência da migração | O script da API recusou os marcadores com versão ou SHA ausentes ou inválidos antes de `update-service`. |
| Credenciais no build da UI | O YAML separa o build com leitura do job com OIDC e verifica o artefato antes de assumir a role. A regressão na avaliação de contexto impede que o fluxo chegue a esses passos no GitHub. |

## Verificações locais

| Área | Comando ou método | Saída | Resultado |
| --- | --- | --- | --- |
| API · SDK 10.0.401 | `dotnet restore ScrapBook.sln --locked-mode` | 0 | Sem warnings. |
| API · compilação | `dotnet build ScrapBook.sln --no-restore --no-incremental --disable-build-servers` | 0 | Zero erros e warnings. |
| API · estilo | `dotnet format ScrapBook.sln --verify-no-changes --no-restore --severity info` | 0 | Relatório de diagnósticos vazio. |
| API · testes | `dotnet test ScrapBook.sln --no-build --no-restore` | 0 | 45 passaram. |
| UI · Node 26.13.0 e npm 11.20.0 | `npm run lint`, `npm run build` e `npm test` em execuções independentes | 0 em cada | Sem warnings do Oxlint, TypeScript e Vite. Dez testes passaram. |
| Liquibase · PostgreSQL 18.6 descartável | `python3 -B tests/test_baseline.py` | 0 | 20 passaram. |
| Reserva · remoto Git descartável | `python3 -B -m unittest discover -s tests -v` | 0 | Cinco passaram. O Git emitiu cinco ocorrências do aviso `You appear to have cloned an empty repository.` durante a preparação da fixture, sem linha de código associada. |
| Verificador HTTPS | `python3 -B -m unittest discover -s deploy -p 'test_*.py' -v` | 0 | Quatro passaram com respostas sintéticas. |
| Publicação | `python3 -B deploy/verify_workflows.py` com os três checkouts da história | 0 | 27 cenários simulados passaram. |
| Sintaxe e contextos | `git diff --check`, leitura YAML, `bash -n`/`sh -n` e comparação com a tabela de contextos do GitHub | 1 na verificação de contextos | Nove YAMLs e treze scripts shell passaram na análise de sintaxe. Um contexto inválido no `env` de job. |

A revisão usou o diff local sobre os commits `d912641` (infraestrutura), `537e725` (API), `7dc6525` (UI), `6a05707` (testes), `fb4cbbb` (docs) e `3b181b1` (Liquibase). Incluiu os dois novos scripts da UI no checkout. O parecer anterior permanece como evidência do estado revisado antes das correções. [Primeira revisão](f01-story-003.md)

Cobertura de linhas e branches ficou sem medição. A execução no GitHub Actions e na AWS segue pendente.
