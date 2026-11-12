# Boas práticas para criação de scripts SQL

[Índice](README.md) · [Liquibase](liquibase.md) · [Backend](backend.md) · [HLD](../HLD.md) · [Modelagem proposta](../db/README.md)

Definição: **2026-11-11**. Este guia trata da escrita e revisão de SQL PostgreSQL; o [guideline de Liquibase](liquibase.md) define versionamento, empacotamento e execução. Exemplos são didáticos e não criam o modelo físico do produto.

As referências consultadas são da documentação PostgreSQL 18. Isso não fixa a versão do RDS: conferir compatibilidade antes de adotar recursos específicos.

## 1. Regras obrigatórias

- Toda tabela possui uma PK simples chamada `id`, inteira e gerada pelo banco.
- Campos comuns usam nomes sem prefixos de tabela ou tipo; FKs, constraints nomeáveis e índices recebem nomes compostos.
- Cada campo deve ter tipo, tamanho, nulabilidade e limites justificados pelo dado.
- Todo script de avanço e rollback deve admitir repetição sem duplicar efeitos ou desfazer dados alheios.
- Existência não comprova compatibilidade: verificar o estado encontrado e interromper diante de divergência.
- Toda alteração deve entregar rollback. Quando não for possível identificar uma reversão viável, preparar o impacto e as alternativas e **questionar a necessidade da alteração ao responsável**.

Essa regra de PK substitui a orientação inicial de UUID como chave relacional. UUID continua permitido para dados cuja semântica o exige, como correlação, mensagens e chaves idempotentes; esses valores não substituem o `id` inteiro da tabela.

## 2. Nomenclatura

Usar inglês, minúsculas, `snake_case`, sem acentos, aspas delimitadoras ou palavras reservadas. Tabelas no plural; objetos referenciados com schema explícito. Escrever palavras-chave SQL em maiúsculas e usar quatro espaços de indentação.

| Objeto | Convenção | Exemplo |
| --- | --- | --- |
| Tabela | Nome do conjunto, sem `tb_`. | `events.events`, `records.files`. |
| PK — coluna | `id`. | `id`, sem `event_id` na própria tabela events. |
| Campo comum | Nome que descreve o dado, sem prefixo de tipo/tabela. | `name`, `email`, `created_at`, `size`; evitar `str_name` e `event_name` dentro de events. |
| Coluna FK | `<tabela_origem>_<tabela_destino>_id`. | Em records, `records_events_id` referencia `events.id`. |
| FK com papéis distintos | `<origem>_<papel>_<destino>_id`. | `records_author_accounts_id`, `records_reviewer_accounts_id`. |
| Constraint PK | `pk_<tabela>_id`. | `pk_events_id`. |
| Constraint FK | `fk_<origem>_<destino>_id`, incluindo papel quando necessário. | `fk_records_events_id`. |
| Constraint UNIQUE | `uq_<tabela>_<campos_ou_regra>`. | `uq_accounts_email`. |
| Constraint CHECK | `ck_<tabela>_<regra>`. | `ck_files_size_positive`. |
| Índice | `ix_<tabela>_<campos_ou_finalidade>`. | `ix_records_events_id`, `ix_records_received_at_id`. |

Uma **coluna FK**, sua **constraint FK** e o **índice de suporte** são objetos diferentes. Nomear cada um. Campos compostos semanticamente, como `created_at`, continuam válidos; a proibição é de prefixos redundantes.

Para autorreferência, explicitar o papel, por exemplo `events_parent_events_id`. Para FK composta, nomear a relação e identificar todas as colunas no DDL, mantendo a PK simples `id`.

Essas convenções são decisões do projeto. O PostgreSQL limita identificadores a **63 bytes** na configuração padrão; abreviar nomes longos de forma estável e documentada, preservando unicidade. Não depender do truncamento automático. [Referência: identificadores](https://www.postgresql.org/docs/18/sql-syntax-lexical.html).

## 3. Chaves primárias e relacionamentos

Padrão para novas tabelas:

```sql
id bigint GENERATED ALWAYS AS IDENTITY,
CONSTRAINT pk_example_events_id PRIMARY KEY (id)
```

`bigint` é um número inteiro de 64 bits. O padrão uniforme acomoda tabelas de crescimento contínuo e evita diferenças de tipo entre PKs e FKs. Uma necessidade de `integer` deve ser justificada pela capacidade esperada e manter os relacionamentos compatíveis.

- Declarar `PRIMARY KEY` explicitamente: identity gera valores, mas não garante unicidade sozinha.
- Deixar o banco gerar o ID; obter o valor com `INSERT ... RETURNING id`.
- Não usar `MAX(id) + 1`, contagem de linhas ou sequência implementada na aplicação.
- Não mudar ou reutilizar IDs para preencher lacunas. Unicidade é por tabela, não uma numeração global do banco.
- FKs usam o mesmo tipo da chave referenciada e referenciam PK ou chave candidata UNIQUE.
- Tabelas associativas também possuem `id`; adicionar UNIQUE sobre o conjunto de FKs para impedir vínculos duplicados.
- IDs sequenciais não autorizam acesso. O backend verifica vínculo e permissão independentemente do identificador.
- No .NET, mapear `bigint` para `long`. Quando exposto em JSON, usar string decimal para preservar o alcance no JavaScript; o consumidor trata o ID como opaco.

As sequências podem ter lacunas por rollback ou conflito de inserção. Idempotência não exige restaurar seu contador. [Identity](https://www.postgresql.org/docs/18/ddl-identity-columns.html), [sequências](https://www.postgresql.org/docs/18/functions-sequence.html).

## 4. Tipagem, tamanho e nulabilidade

| Necessidade | Escolha e verificação |
| --- | --- |
| Inteiros | `smallint`, `integer` ou `bigint` conforme faixa, crescimento e relacionamento. Não usar texto para cálculo. |
| Decimal exato | `numeric(p,s)`, com precisão, escala e faixa explícitas. Não usar ponto flutuante para valores que exigem exatidão. |
| Medida aproximada | `real`/`double precision` somente quando aproximação for aceitável e documentada. |
| Booleano | `boolean`; não `0/1` ou texto livre. |
| Texto limitado pelo domínio | `varchar(n)`, com `n` derivado do requisito. Legenda: até 500; depoimento: até 5.000 caracteres. |
| Texto sem limite de negócio conhecido | `text`, definindo os limites de entrada antes de publicar o contrato. Não inventar `varchar(255)` por hábito. |
| Código de comprimento fixo | `char(n)` somente se o preenchimento com espaços fizer parte da regra. |
| Data civil | `date`, sem conversão artificial para instante. |
| Data e hora de negócio | `timestamp without time zone`, interpretado em `America/Sao_Paulo`, conforme o HLD. |
| Instante técnico/externo | Representação e conversão explícitas; usar `timestamptz` quando a semântica for um instante. Não reinterpretar UTC como Brasília. |
| Duração/tamanho | Tipo numérico e unidade documentados, como segundos ou bytes; validar mínimo/máximo. |
| Identificador externo | Tipo/formato do contrato do provedor; não transformar todo identificador em inteiro ou UUID. |
| Metadados estruturados | `jsonb` quando houver necessidade real; relações e campos essenciais continuam tipados. |
| Conteúdo de mídia | Arquivo no S3; banco armazena referências e metadados, conforme o HLD. |

Faixas dos inteiros: smallint até 32.767 positivos; integer até 2.147.483.647; bigint até 9.223.372.036.854.775.807. `numeric` pode arredondar ao ajustar escala: validar o dado antes de gravar. [Tipos numéricos](https://www.postgresql.org/docs/18/datatype-numeric.html).

O tamanho de `varchar(n)` é em caracteres, não bytes, e não reserva sempre `n` posições. Evitar casts que truncam silenciosamente e `char(n)` por suposta vantagem de desempenho. Alinhar a contagem de caracteres com frontend/API, inclusive para Unicode; validar bytes separadamente quando o contrato exigir. [Tipos de texto](https://www.postgresql.org/docs/18/datatype-character.html).

Definir `NOT NULL` para obrigatoriedade. Não substituir ausência por zero, string vazia ou data fictícia. DEFAULT deve expressar uma regra real; timestamps locais precisam de conversão explícita, sem depender do fuso da conexão. [Datas e horários](https://www.postgresql.org/docs/18/datatype-datetime.html).

## 5. Integridade e índices

- Dar nomes compostos às PKs, FKs, UNIQUEs e CHECKs. Declarar `NOT NULL` diretamente na coluna, observando as capacidades de nomenclatura da versão escolhida.
- Combinar CHECK com nulabilidade: um CHECK de faixa pode aceitar NULL.
- Definir regras de unicidade e a semântica de NULL, maiúsculas/minúsculas e normalização antes de criar a constraint.
- Usar CHECK para a própria linha. Regras entre linhas, como até dois responsáveis, precisam de desenho transacional/constraints apropriadas.
- Definir `ON DELETE`/`ON UPDATE` pelo ciclo de vida; preservar referências e lembranças confirmadas. Não aplicar CASCADE indiscriminadamente.
- Manter isolamento por evento nas relações. Uma FK simples prova a existência do pai; relações de escopo podem precisar de chaves candidatas compostas e autorização complementar.
- PK e UNIQUE já criam índice de suporte; FK não cria automaticamente índice no lado filho. Justificar cada índice pela consulta, integridade ou operação referencial.
- Não duplicar um índice coberto por outro adequado. Conferir ordem das colunas, filtros, ordenação e desempate.

[Referência: constraints](https://www.postgresql.org/docs/18/ddl-constraints.html).

## 6. Idempotência com validação de estado

Aplicar o script novamente deve preservar o resultado pretendido, sem duplicar linhas, objetos ou transformações. São duas proteções complementares: histórico do Liquibase e SQL preparado para repetição. Isso não autoriza executar DDL manualmente nos ambientes AWS.

| Operação | Proteção obrigatória |
| --- | --- |
| CREATE TABLE/INDEX | Verificar existência e compatibilidade de definição; nome igual não basta. |
| ADD COLUMN/CONSTRAINT | Verificar coluna/constraint na tabela correta, tipo, tamanho, nulabilidade e definição. |
| INSERT | Chave de negócio estável com UNIQUE; guarda de existência e/ou ON CONFLICT direcionado. |
| UPDATE | WHERE por chave e estado esperado; não reaplicar incremento, conversão ou timestamp a cada execução. |
| DELETE | WHERE restrito, origem conhecida, análise de dependências e estratégia para restaurar os dados. |
| DROP/rollback | Verificar existência, propriedade do efeito e dependências; não apagar objeto preexistente apenas porque está presente. |

Ao encontrar um objeto existente, consultar `information_schema`/`pg_catalog`: schema, tabela, colunas, identity, PK, FKs, constraints e índices necessários. Para constraints, restringir a consulta à relação correta; para índices, verificar definição e validade. Se houver incompatibilidade, falhar com erro claro.

`CREATE TABLE IF NOT EXISTS` **não compara a estrutura existente** com a solicitada. Um índice de mesmo nome também não comprova a definição esperada. Não usar guardas como forma de esconder drift. [CREATE TABLE](https://www.postgresql.org/docs/18/sql-createtable.html), [CREATE INDEX](https://www.postgresql.org/docs/18/sql-createindex.html).

### IF no PostgreSQL

Usar `DO $migration$ BEGIN ... IF ... THEN ... END IF; END $migration$;` ou função PL/pgSQL. O formato `IF (...) BEGIN ... END` isolado do exemplo inicial não é a sintaxe PostgreSQL. Preferir `SELECT 1` em EXISTS. O BEGIN dentro de DO delimita o bloco PL/pgSQL; não abre uma transação SQL adicional. [DO](https://www.postgresql.org/docs/18/sql-do.html), [condicionais](https://www.postgresql.org/docs/18/plpgsql-control-structures.html#PLPGSQL-CONDITIONALS).

Verificar existência e depois inserir são etapas sujeitas a corrida. Proteger a chave de negócio com UNIQUE e usar `ON CONFLICT (campos) DO NOTHING` quando a repetição deva manter o registro existente. Conferir os valores depois; não ignorar conteúdo divergente ou usar DO UPDATE para sobrescrevê-lo sem uma regra explícita. [INSERT/ON CONFLICT](https://www.postgresql.org/docs/18/sql-insert.html).

Mudanças concorrentes mais complexas exigem locks/isolamento apropriados. Serializar DDL pelo Liquibase; um IF não coordena dois migradores independentes.

## 7. Exemplos de avanço e rollback

Os exemplos pressupõem o schema `events` existente, execução transacional e validação prévia do contrato dos objetos. Os nomes `example_*` e o código `sql_guideline_demo` são exclusivos desta demonstração. **Em migrations reais, comprovar e registrar se o objeto/linha foi criado pela alteração ou já existia.** Não adotar um objeto alheio pelo simples uso de IF NOT EXISTS.

### Criação de tabelas

```sql
CREATE TABLE IF NOT EXISTS events.example_events (
    id bigint GENERATED ALWAYS AS IDENTITY,
    code varchar(40) NOT NULL,
    name varchar(120) NOT NULL,
    scheduled_at timestamp without time zone NOT NULL,
    CONSTRAINT pk_example_events_id PRIMARY KEY (id),
    CONSTRAINT uq_example_events_code UNIQUE (code),
    CONSTRAINT ck_example_events_name_not_blank CHECK (btrim(name) <> '')
);

CREATE TABLE IF NOT EXISTS events.example_items (
    id bigint GENERATED ALWAYS AS IDENTITY,
    example_items_example_events_id bigint NOT NULL,
    caption varchar(500),
    CONSTRAINT pk_example_items_id PRIMARY KEY (id),
    CONSTRAINT fk_example_items_example_events_id
        FOREIGN KEY (example_items_example_events_id)
        REFERENCES events.example_events (id) ON DELETE RESTRICT
);

CREATE INDEX IF NOT EXISTS ix_example_items_example_events_id
    ON events.example_items (example_items_example_events_id);
```

Após a guarda, validar as definições esperadas; a reaplicação não dispensa essa verificação. O tamanho 120 do nome é ilustrativo, não um novo limite do produto.

Rollback da criação, **somente com proveniência comprovada e tabelas vazias**:

```sql
DO $rollback$
BEGIN
    IF to_regclass('events.example_events') IS NOT NULL THEN
        LOCK TABLE events.example_events IN ACCESS EXCLUSIVE MODE;
    END IF;

    IF to_regclass('events.example_items') IS NOT NULL THEN
        LOCK TABLE events.example_items IN ACCESS EXCLUSIVE MODE;
        IF EXISTS (SELECT 1 FROM events.example_items) THEN
            RAISE EXCEPTION 'Rollback exige tratamento dos dados de example_items.';
        END IF;
        DROP TABLE events.example_items;
    END IF;

    IF to_regclass('events.example_events') IS NOT NULL THEN
        IF EXISTS (SELECT 1 FROM events.example_events) THEN
            RAISE EXCEPTION 'Rollback exige tratamento dos dados de example_events.';
        END IF;
        DROP TABLE events.example_events;
    END IF;
END;
$rollback$;
```

O lock antecede a verificação de tabela vazia para impedir novas gravações entre a checagem e o DROP. Planejar ordem de locks e tempo de espera. O índice e a constraint próprios da tabela são removidos com ela; dependências externas devem impedir o DROP. Não acrescentar CASCADE para contorná-las. [DROP TABLE](https://www.postgresql.org/docs/18/sql-droptable.html).

### Inserção de referência

Avanço, sobre a tabela demonstrativa anterior:

```sql
DO $migration$
BEGIN
    IF NOT EXISTS (
        SELECT 1
        FROM events.example_events
        WHERE code = 'sql_guideline_demo'
    ) THEN
        INSERT INTO events.example_events (code, name, scheduled_at)
        VALUES ('sql_guideline_demo', 'Evento de exemplo', TIMESTAMP '2027-01-20 18:00:00')
        ON CONFLICT (code) DO NOTHING;
    END IF;

    IF NOT EXISTS (
        SELECT 1
        FROM events.example_events
        WHERE code = 'sql_guideline_demo'
          AND name = 'Evento de exemplo'
          AND scheduled_at = TIMESTAMP '2027-01-20 18:00:00'
    ) THEN
        RAISE EXCEPTION 'O código de referência existe com dados incompatíveis.';
    END IF;
END;
$migration$;
```

Rollback, condicionado à posse da linha pelo changeset e ao seu estado esperado:

```sql
DO $rollback$
BEGIN
    DELETE FROM events.example_events
    WHERE code = 'sql_guideline_demo'
      AND name = 'Evento de exemplo'
      AND scheduled_at = TIMESTAMP '2027-01-20 18:00:00';

    IF EXISTS (
        SELECT 1
        FROM events.example_events
        WHERE code = 'sql_guideline_demo'
    ) THEN
        RAISE EXCEPTION 'A referência mudou; revisar o rollback antes de removê-la.';
    END IF;
END;
$rollback$;
```

Se um INSERT foi ignorado porque a linha já existia antes da migration, não removê-la no rollback. O código exclusivo acima é uma precondição da demonstração, não substitui a prova de origem em dados reais.

### ALTER, UPDATE e DELETE

- Em ADD COLUMN, usar guarda e confirmar tipo/tamanho/nulabilidade após a operação. O rollback só remove uma coluna criada por essa alteração; se ela já recebeu dados, avaliar preservação antes de remover.
- Para ADD CONSTRAINT, consultar `pg_constraint` pela tabela e nome e comparar a definição. Não há um `ADD CONSTRAINT IF NOT EXISTS` genérico para substituir esse controle.
- Em UPDATE, guardar os valores anteriores quando necessários ao rollback. Usar chave e estado anterior esperado; `IS DISTINCT FROM` ajuda a comparar valores com NULL e evitar atualizações sem mudança.
- Em DELETE, preservar antes os registros e relações necessários à restauração. Não oferecer um INSERT com valores inventados como rollback.
- Reaplicar rollback também deve ser seguro: estado já revertido é no-op; estado divergente exige diagnóstico, não sucesso silencioso.

[ALTER TABLE](https://www.postgresql.org/docs/18/sql-altertable.html), [comparações com NULL](https://www.postgresql.org/docs/18/functions-comparison.html).

## 8. Rollback obrigatório e exceções

Cada arquivo de avanço em `sql/` deve possuir seu correspondente em `rollback/`, registrado no changelog. Reverter dependências na ordem inversa. Para funções, views ou triggers substituídos, restaurar a definição anterior quando aplicável.

Registrar antes de executar:

1. Estado inicial e quais objetos/linhas a alteração realmente criará ou modificará.
2. Evidência persistente necessária para reconhecer esses efeitos em outra execução.
3. Valores anteriores e dados necessários para recuperação.
4. Efeitos de novas gravações após a aplicação e compatibilidade da versão anterior.
5. Pré-condições, dependências e validações do rollback.

`DATABASECHANGELOG` registra a execução do changeset; não prova que um CREATE protegido criou a tabela ou que um INSERT com ON CONFLICT inseriu a linha. Registrar essa distinção quando ela influenciar a reversão.

**Quando não identificar rollback viável, questionar a necessidade antes de executar a alteração ou declará-la concluída.** Preparar uma explicação concreta: dado perdido, impacto, alternativas de expansão/contração, preservação de valores anteriores e opções de recuperação.

Exemplo de pergunta para esse caso: “Esta alteração elimina X e não identifiquei como restaurar Y com segurança. Ela é necessária? Podemos preservar esses dados ou aprovar uma exceção com plano de recuperação?”

Somente uma exceção explicitamente aceita permite omitir o SQL de rollback. Registrar motivo, responsável, decisão e recuperação possível. Não criar rollback vazio nem apresentar backup genérico ou migração corretiva como reversão já comprovada.

Rollback de banco continua manual e pelo workflow Liquibase. Um DROP pode ser inverso do CREATE e ainda destruir dados recebidos depois; essa análise faz parte da viabilidade.

## 9. Execução, segurança e desempenho

- Uma intenção coesa por script; cabeçalho com objetivo, dependências, pré/pós-condições, impacto e caminho do rollback.
- Qualificar tabelas por schema; não depender de `search_path` para atingir o objeto correto.
- Listar colunas em INSERT e usar chaves estáveis em buscas de dados de referência.
- Parametrizar valores externos; identificadores dinâmicos exigem lista permitida e escape apropriado, como `format('%I', nome)`.
- Não incluir credenciais, conteúdo real de convidados ou dumps sensíveis em scripts/logs.
- Usar identidade de migração com os privilégios necessários, distinta da identidade DML da aplicação; evitar superusuário como padrão.
- Dados de negócio usam tabelas permanentes com logging. TEMP/UNLOGGED exigem finalidade descartável explícita e não podem armazenar lembranças confirmadas: UNLOGGED não oferece a mesma segurança após crash. [CREATE TABLE](https://www.postgresql.org/docs/18/sql-createtable.html).
- Planejar transação curta, ordem de locks, volume e limites de espera/execução. No Liquibase transacional, não inserir BEGIN/COMMIT SQL próprios.
- Para DO/PL/pgSQL, configurar divisão de instruções compatível, normalmente um bloco por arquivo com `splitStatements: false`, validado no executor real.
- Em tabelas grandes, planejar backfill em lotes, checkpoints, retomada e comparação de contagens/integridade. Evitar uma transação longa sem dimensionamento.
- Considerar índices concorrentes quando necessários; isolar o comando não transacional e tratar índices inválidos após falha. IF NOT EXISTS não corrige índice inválido.
- Avaliar plano com volume representativo. EXPLAIN ANALYZE executa a instrução; usá-lo em ambiente adequado e tratar escritas como escritas.
- Não mudar SQL já aplicado para corrigir seu efeito. Criar uma nova migration, preservando o histórico e checksums.

[Locks](https://www.postgresql.org/docs/18/explicit-locking.html), [índices concorrentes](https://www.postgresql.org/docs/18/sql-createindex.html).

## 10. Revisão e validação

- [ ] Todas as tabelas possuem `id` inteiro identity e PK nomeada.
- [ ] Campos comuns não têm prefixos redundantes; FKs, constraints e índices têm nomes compostos.
- [ ] Tipos, tamanho, unidades, precisão, nulabilidade e defaults refletem os requisitos.
- [ ] FKs, UNIQUEs, CHECKs e índices protegem integridade sem duplicação desnecessária.
- [ ] Avanço foi executado duas vezes em base isolada, sem duplicar dados ou efeitos.
- [ ] Estado preexistente compatível e incompatível foi testado; divergências falham explicitamente.
- [ ] Concorrência relevante foi testada; IF não é a única proteção de unicidade.
- [ ] Objetos/dados preexistentes não são apagados indevidamente pelo rollback.
- [ ] Rollback, repetição do rollback e nova aplicação foram ensaiados.
- [ ] UPDATE/DELETE preservam os dados necessários; contagens e relações foram conferidas.
- [ ] Perda de dados, locks, volume e compatibilidade de deploy foram avaliados.
- [ ] Falta de reversão viável foi questionada e eventual exceção ficou explicitamente registrada.

Esses itens são critérios para os scripts futuros. A publicação deste guideline não equivale à execução dos exemplos ou à criação de migrations.
