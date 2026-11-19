# Code review da F01-STORY-001

**Parecer da revisão inicial. Solicitar alterações antes do push.**

Revisão dos candidatos `698b95a` de `scrapbook-liquibase` e `876790e` de `scrapbook-docs`, comparados às bases `origin/dev` `fc0b2d1` e `43d6685` respectivamente. Os critérios são os da instalação, reaplicação, permissões e reversão da história. [Trello](https://trello.com/c/gSGvlzEZ) · [Spec da F01](../spec.md#6-modelo-de-dados)

## Achados

### P1. Privilégios preexistentes podem dar acesso à auditoria e ao histórico do Liquibase

O bootstrap confere os atributos das roles e a associação com `scrapbook_owner`, mas não confere outras associações privilegiadas nem defaults de tabelas criadas pela proprietária. Em duas sondagens descartáveis, o bootstrap e a instalação aceitaram o alvo e a identidade da aplicação obteve SELECT em `audit_entries` e `databasechangelog`. Uma sondagem partiu de um default de SELECT para PUBLIC. A outra partiu de uma associação preexistente a uma role de leitura ampla que a aplicação podia assumir. A assinatura inicial registrou o catálogo sem exigir a matriz efetiva de privilégios da história.

Validar as concessões padrão e os papéis assumíveis previstos na matriz antes de aceitar o bootstrap. Conferir os privilégios efetivos nas tabelas de controle e na auditoria antes de confirmar a instalação. Cobrir esses dois estados preexistentes na suíte. Achado introduzido neste candidato e reproduzido localmente. [Bootstrap, linhas 35 e 49–52](../../../scrapbook-liquibase/config/bootstrap.sql) · [Guideline de Liquibase](../../guidelines/liquibase.md#ownership-grants-e-permissões-padrão)

### P2. O checkpoint inicial bloqueia o segundo update de uma evolução válida

O changelog principal compara o catálogo com a assinatura registrada pela instalação inicial. Um changeset adicional sintético alterou o catálogo e foi aplicado com sucesso. O mesmo `update`, repetido sobre o mesmo changelog, falhou com “divergência no catálogo”, com os dois changesets registrados. A base atual não estabelece como gravar e restaurar a assinatura de uma nova geração.

Versionar a verificação e a transição do checkpoint com a evolução do schema. Cobrir aplicação, reaplicação e reversão de um changeset posterior sem apagar o histórico. Achado introduzido neste candidato e reproduzido localmente. [Precondition de catálogo, linha 43](../../../scrapbook-liquibase/db.changelog-master.yaml) · [Checkpoint inicial, linha 55](../../../scrapbook-liquibase/changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql)

### P2. A assinatura ignora alterações na definição das sequências identity

A assinatura inclui o nome e metadados genéricos da sequência, porém omite os parâmetros em `pg_sequence`. Em banco vazio, mudar o incremento da sequência identity de `accounts` para 10 deixou a assinatura idêntica. O `update` e o rollback terminaram com sucesso, embora o catálogo da sequência estivesse diferente do instalado.

Incluir os parâmetros de definição das sequências, como incremento e limites, na verificação de catálogo. Deixar a posição corrente fora da assinatura para permitir avanço por uso normal. Cobrir a recusa de update e rollback quando a definição mudar. Achado introduzido neste candidato e reproduzido localmente. [Assinatura de relações, linha 18](../../../scrapbook-liquibase/changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql) · [Guarda de rollback, linha 35](../../../scrapbook-liquibase/changelogs/sprint-1/rollback/001-remover-base-inicial.sql)

## Verificações

| Verificação | Resultado |
| --- | --- |
| `git diff origin/dev..<SHA> --check` nos dois repositórios | Código 0, sem diagnósticos de whitespace. |
| Sintaxe de `tests/test_baseline.py` | Validação por `ast.parse`, código 0. |
| Suíte original e sondagens adicionais | 12 cenários em PostgreSQL 18.6 descartável com Liquibase 4.33.0. Os 8 cenários originais passaram. As 4 sondagens adicionais confirmaram os comportamentos relatados acima. |
| Lint e build de frontend/backend | Indisponíveis nas revisões locais dos componentes, que contêm somente a base documental e arquivos Git. |
| Ambientes publicados | Fora do alcance da revisão local. |

Os cenários originais confirmam o inventário de 24 tabelas, o rollback em banco vazio e a recusa de reversão com dados ou dependências externas. O aceite de permissões e reaplicação fica condicionado aos achados acima. [Evidência da implementação](../dev-db/validacao-liquibase.md)

## Reteste das correções

Os três achados foram corrigidos e revalidados em PostgreSQL 18.6 descartável com Liquibase 4.33.0. Os 14 testes passaram. O preparo valida associações e defaults, o checkpoint confere os privilégios efetivos e registra a geração de cada changeset de catálogo, e a assinatura distingue a definição da sequência do seu valor corrente. [Evidência do reteste](../dev-db/validacao-liquibase.md#revalidação-após-o-code-review)

## Segunda revisão

**Parecer. Solicitar alterações antes do push.** Candidatos `3bf9f0a` no Liquibase e `8931c47` na documentação. A revisão manteve as mesmas bases `origin/dev` e executou sondagens adicionais em PostgreSQL 18.6 descartável com Liquibase 4.33.0.

### P1. O Liquibase deixa as tabelas de controle com acesso indevido após recusar a instalação

O bloqueio de privilégios efetivos ocorre na última etapa do changeset, depois de o próprio Liquibase criar `databasechangelog` e `databasechangeloglock`. Em uma sondagem, um default inadequado de SELECT e UPDATE foi configurado depois do bootstrap. A instalação falhou, as tabelas de aplicação foram revertidas e o histórico ficou vazio, mas a role `scrapbook_app` manteve SELECT e UPDATE nas duas tabelas de controle. A verificação versionada desse cenário confirma as 24 tabelas de aplicação ausentes e o histórico vazio. Ela não verifica os privilégios remanescentes.

Verificar os defaults antes da criação dos controles e assegurar a matriz efetiva também nos controles já existentes. Acrescentar a pós-condição de que uma instalação recusada não deixa esses objetos acessíveis à aplicação. Achado residual reproduzido localmente. [Ordem do changeset, linha 38](../../../scrapbook-liquibase/changelogs/sprint-1/changelog.yaml) · [Validação de privilégios, linha 97](../../../scrapbook-liquibase/changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql) · [Teste do alvo recusado, linha 185](../../../scrapbook-liquibase/tests/test_baseline.py)

### P2. A reversão pode aceitar um catálogo diferente como a geração anterior

`checkpoint_catalog` valida o nome da geração de origem e grava o hash corrente sob o nome da geração de destino. Em um changeset sintético, o rollback removeu apenas uma das duas colunas que o update criou. O Liquibase aceitou a reversão, gravou o catálogo residual como `f01-story-001-base` e aceitou um `update` da base. A aplicação seguinte da evolução falhou porque a segunda coluna ainda existia.

Preservar a assinatura original de cada geração e exigir que o catálogo revertido corresponda a ela antes de confirmar o rollback. O teste de uma reversão incompleta deve verificar erro, preservação da geração vigente e histórico intacto. Achado reproduzido localmente no protocolo introduzido pela correção. [Gravação do checkpoint, linha 101](../../../scrapbook-liquibase/changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql) · [Precondition de reaplicação, linha 46](../../../scrapbook-liquibase/db.changelog-master.yaml)

Os 14 testes versionados passaram. As duas sondagens acima confirmaram os problemas adicionais. `git diff --check` passou nos dois candidatos. O lint e o build de frontend e backend seguem indisponíveis porque esses repositórios ainda não têm arquivos de projeto.

## Reteste da segunda revisão

Os dois achados passaram no reteste local. O comando de migração verifica privilégios por JDBC antes de iniciar a CLI. A reversão compara o catálogo resultante com a assinatura histórica do destino dentro da transação do changeset. A suíte completa passou com 15 testes, incluindo a recusa da instalação antes de criar os controles e a preservação do histórico após uma tentativa de rollback incompleto. [Evidência do reteste](../dev-db/validacao-liquibase.md#revalidação-após-a-segunda-revisão)

## Terceira revisão

**Parecer. Solicitar alterações antes do push.** Candidatos `dbdbf6f` no Liquibase e `318c2fe` na documentação, na branch `feature/F01-STORY-001`. As bases observadas continuam `origin/dev` `fc0b2d1` e `43d6685`. A revisão usou a spec da F01 disponível no diretório de trabalho e verificou o provisionamento, os controles do Liquibase, a assinatura do catálogo, a instalação e a reversão. A spec e o plano seguem como arquivos locais, com alterações locais já existentes no PRD e no índice de documentação. [Spec da F01](../spec.md#6-modelo-de-dados)

### P2. A assinatura aceita uma tabela de auditoria sem WAL

A assinatura da relação omite `pg_class.relpersistence`. Em um PostgreSQL descartável, mudar `audit_entries` de permanente para `UNLOGGED` manteve a mesma assinatura. Tanto o `update` quanto o rollback da base vazia terminaram com código 0. A reaplicação pode confirmar um estado em que os registros da auditoria podem ser perdidos após uma parada abrupta. A lacuna vem da assinatura original e também afeta a comparação com assinaturas históricas. Incluir a persistência das relações na assinatura e verificar que a mudança bloqueia a reaplicação e a reversão. [Assinatura de relações, linha 29](../../../scrapbook-liquibase/changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql#L29) · [Integridade da auditoria](../../guidelines/observabilidade.md#integridade-e-correção-da-auditoria)

### P2. A instalação aceita histórico de outro changelog

O ramo de instalação verifica a ausência do changeset inicial e de objetos gerenciados. Ele aceita outros registros no histórico. Em banco descartável, o próprio Liquibase aplicou um changelog sintético sem DDL. A instalação da F01 terminou com código 0 e manteve os dois registros em `databasechangelog`. Isso permite adotar um alvo de origem desconhecida sem a transição exigida pelo plano. Exigir histórico vazio para instalar a base ou validar uma transição explícita. A lacuna existe desde o changelog inicial. [Verificação de instalação, linha 19](../../../scrapbook-liquibase/db.changelog-master.yaml#L19) · [Pré-requisitos do plano](../plan.md#pré-requisitos)

### P2. A pré-validação aceita acesso ao histórico fora da matriz

A pré-validação de controles consulta apenas `scrapbook_app` e `scrapbook_audit_reader`. Em banco descartável, uma role sintética recebeu `USAGE` no schema antes da instalação e, depois, recebeu `SELECT` em `databasechangelog`. O PostgreSQL confirmou ambos os privilégios e o `update` terminou com código 0. A matriz do repositório reserva os controles à proprietária e à sessão de migração. Conferir os grants de tabela e coluna dos controles para os beneficiários autorizados. A omissão já existia na checagem efetiva da assinatura e continua no fluxo introduzido em `dbdbf6f`. [Pré-validação, linha 39](../../../scrapbook-liquibase/config/preflight.sql#L39) · [Matriz documentada, linha 9](../../../scrapbook-liquibase/README.md#L9)

| Verificação | Resultado |
| --- | --- |
| `python3 tests/test_baseline.py` | Código 0. Os 15 testes passaram em PostgreSQL 18.6 descartável com Liquibase 4.33.0. A recusa antes de criar os controles e a rejeição do rollback que deixa uma coluna residual foram confirmadas. |
| Sondagens locais adicionais | Os três casos acima foram reproduzidos em PostgreSQL descartável. |
| `git diff origin/dev...<SHA> --check` | Código 0 e sem diagnósticos nos dois candidatos. |
| `ast.parse` para os dois arquivos Python e `javac -Xlint:all` para `MigrationPreflight.java` | Código 0 nas duas verificações. O compilador Java não emitiu warnings. |
| Lint e build de frontend e backend | Indisponíveis. As cópias locais contêm apenas a base documental e arquivos Git. |

A execução cobre o banco descartável. Pipelines e ambientes publicados ficam fora do escopo desta história.

## Revisão técnica com a skill atualizada

**Parecer. Corrigir os três achados técnicos P2 antes do push.** O código permanece no commit `dbdbf6f`, comparado à base `origin/dev` `fc0b2d1`. A documentação está no commit `318c2fe`, com as alterações locais já registradas na terceira revisão. Esta rodada verificou as preconditions, a assinatura do catálogo, o rollback, o launcher JDBC e as permissões dos controles. A spec e o plano serviram de referência para o estado técnico esperado da migração. [Guideline de Liquibase](../../guidelines/liquibase.md) · [Guideline de SQL](../../guidelines/sql.md)

| Achado técnico confirmado | Evidência desta rodada | Ajuste |
| --- | --- | --- |
| P2. Persistência fora da assinatura | A alteração de `audit_entries` para `UNLOGGED` manteve a assinatura. A reaplicação e o rollback da base vazia retornaram código 0. | Incluir a persistência das relações na assinatura. [Código](../../../scrapbook-liquibase/changelogs/sprint-1/sql/007-assinatura-do-catalogo.sql#L29) · [Validação de estado](../../guidelines/sql.md#6-idempotência-com-validação-de-estado) |
| P2. Histórico anterior aceito na primeira instalação | O Liquibase registrou um changelog sintético em um banco sem tabelas de aplicação. O `update` instalou as 24 tabelas e manteve os dois registros no histórico. | Exigir histórico vazio na instalação inicial ou validar uma transição explícita. [Código](../../../scrapbook-liquibase/db.changelog-master.yaml#L19) · [Pré-requisitos](../plan.md#pré-requisitos) |
| P2. Grant nos controles fora da matriz | Uma role sintética com `USAGE` no schema obteve `SELECT` em `databasechangelog`. A reaplicação terminou com código 0. | Conferir os beneficiários dos grants de tabela e coluna dos controles conforme a matriz. [Código](../../../scrapbook-liquibase/config/preflight.sql#L39) · [Ownership e grants](../../guidelines/liquibase.md#ownership-grants-e-permissões-padrão) |

Nenhum achado técnico novo nesta rodada. As origens dos três desvios estão registradas na [terceira revisão](#terceira-revisão).

| Check | Resultado |
| --- | --- |
| `python3 tests/test_baseline.py` | Código 0. Os 15 testes passaram em PostgreSQL 18.6 descartável com Liquibase 4.33.0. |
| Três sondagens técnicas em PostgreSQL descartável | Código 0. Cada sondagem reproduziu o comportamento descrito no achado correspondente. |
| Sintaxe Python com `ast.parse` e compilação Java com `javac -Xlint:all` | Código 0 em ambos. Java 26.0.1, sem warnings de compilação. |
| `git diff origin/dev...<SHA> --check` | Código 0 e sem diagnósticos para Liquibase e documentação. |
| Lint e build de frontend e backend | Indisponíveis pela ausência de projetos nas cópias locais. |

### Reteste dos três achados técnicos

A assinatura passou a incluir o modo de persistência das relações. A instalação inicial exige histórico vazio, e a pré-validação recusa grants de tabela e coluna dos controles para beneficiários fora da matriz. Os 18 testes da suíte passaram em PostgreSQL descartável. [Evidências do reteste](../dev-db/validacao-liquibase.md#revalidação-dos-três-achados-técnicos)

## Revisão técnica do commit 107b5cb

**Parecer. As três regressões P2 passaram. Resta um achado P3 na verificação de grants.** Foram revisados o Liquibase em `107b5cb` e a documentação em `e42254e`, na branch `feature/F01-STORY-001`. As bases locais de comparação são `origin/dev` `fc0b2d1` e `43d6685`. A assinatura rejeitou a mudança para `UNLOGGED`, a instalação recusou o histórico alheio e a pré-validação bloqueou os grants de tabela e coluna de aplicação fora da matriz. [Reteste do desenvolvimento](../dev-db/validacao-liquibase.md#revalidação-dos-três-achados-técnicos)

### P3. A verificação de grants ignora colunas de sistema

A condição `attribute.attnum > 0` restringe a busca de ACLs a colunas de aplicação. Em banco descartável, após uma concessão administrativa explícita de SELECT sobre a coluna de sistema `ctid` de `databasechangelog`, uma role sintética fora da matriz conseguiu ler a posição física da linha e o `update` terminou com código 0. O acesso observado ficou restrito a esse metadado. A lacuna é residual no achado de grants. O filtro foi introduzido em `107b5cb`. Incluir as colunas de sistema na inspeção de ACL para aplicar a matriz a todos os grants de coluna. [Filtro, linha 55](../../../scrapbook-liquibase/config/preflight.sql#L55) · [Ownership e grants](../../guidelines/liquibase.md#ownership-grants-e-permissões-padrão)

| Check | Resultado |
| --- | --- |
| `python3 tests/test_baseline.py` | Código 0. Os 18 testes passaram com PostgreSQL 18.6 descartável e Liquibase 4.33.0. |
| Sondagem de grants | A pré-validação bloqueou SELECT de tabela para PUBLIC e aceitou o grant de coluna de sistema descrito acima. |
| Sintaxe Python com `ast.parse` e Java com `javac -Xlint:all` | Código 0 em ambos. Java 26.0.1, sem warnings de compilação. |
| `git diff origin/dev...HEAD --check` | Código 0 e sem diagnósticos em Liquibase e documentação. |
| Lint e build de frontend e backend | Indisponíveis pela ausência de projetos nas cópias locais. |
