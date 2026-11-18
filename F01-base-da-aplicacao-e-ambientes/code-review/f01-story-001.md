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
