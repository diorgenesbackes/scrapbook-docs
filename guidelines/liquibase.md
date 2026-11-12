# Guideline de Liquibase e PostgreSQL

[Índice](README.md) · [Scripts SQL](sql.md) · [Princípios](principios.md) · [Backend](backend.md) · [Git Flow](git-flow.md) · [HLD](../HLD.md)

Base: PostgreSQL no RDS, Liquibase como responsável exclusivo pelo schema e GitHub Actions executando tarefa Fargate. Em 2026-11-12, a [base inicial](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) passou a conter scripts e rollbacks correspondentes à [modelagem](../db/README.md). Versões e comandos locais estão fixados no componente: Liquibase 4.33.0, PostgreSQL 18.6, Java utilizado 26.0.1 e JDBC 42.7.7. Provisionamento RDS, identidades e pipeline AWS continuam pendentes.

## 1. Responsabilidade e estrutura

Toda mudança estrutural deve ser versionada no repositório Liquibase. A aplicação usa EF Core para acesso/mapeamento; não cria tabelas ou executa migrations no startup.

Escrever o SQL conforme o [guideline exclusivo de scripts](sql.md): PKs inteiras, tipagem, nomes, idempotência com validação de estado e rollback obrigatório. Este guia concentra a orquestração pelo Liquibase.

Estrutura adotada na base inicial:

```text
scrapbook-liquibase/
  db.changelog-master.yaml
  changelogs/
    sprint-1/
      changelog.yaml
      sql/
      rollback/
  config/                     # apenas exemplos públicos; segredos fora do Git
  tests/                      # validações próprias das migrations
  README.md
```

Adotar changelogs YAML, SQL e rollback separados, preservando o formato de origem. Includes explícitos no master, com `relativeToChangelogFile: true`; ordenar por dependência, sem depender de descoberta implícita de diretórios.

Convenção proposta: `NNN-acao-objeto.sql`, rollback correspondente, changeset `sprint-N-NNN`, autor estável `scrapbook` e `dbms: postgresql`. Reservar identificadores para histórias paralelas; o número do ID não define a execução, o changelog define.

Os módulos usam schemas separados no mesmo banco de cada ambiente. A base inicial materializa `identity`, `events`, `records` e `notifications`, conforme o modelo.

## 2. Histórico imutável e identidade

Após aplicação em base compartilhada, não alterar SQL, ID, autor ou caminho para mudar o efeito de um changeset. Criar nova migração corretiva; não reordenar o histórico aplicado.

Identidade e checksum precisam ser preservados. Se mover arquivo aplicado for indispensável, planejar `logicalFilePath` conforme a versão escolhida e testar reconhecimento do histórico em base existente e vazia. Não presumir que um caminho lógico novo reconhecerá automaticamente o anterior.

Não usar limpeza de checksums, `validCheckSum`, edição de tabelas de controle, `runAlways` ou `runOnChange` para esconder divergência. Objetos repetíveis exigem decisão específica; o padrão estrutural é incremental.

Todo SQL de avanço e rollback deve ser preparado para repetição segura, além do controle de execução do Liquibase. Usar guardas de existência e estado, UNIQUE/ON CONFLICT e demais mecanismos conforme a operação. `IF NOT EXISTS` não compara estruturas nem impede toda corrida; divergências precisam causar falha explícita.

## 3. Unidade de mudança e transação

Changeset deve ter intenção coesa e verificável. Tabela com constraints/índice necessário pode ser unidade; alterações com dependências ou riscos distintos devem ser separadas.

Usar `runInTransaction: true` quando a operação permitir. Liquibase controla a transação; não adicionar `BEGIN/COMMIT` ao SQL. Comando que exige execução não transacional deve ficar isolado, com recuperação de falha parcial documentada.

`sqlFile` declara caminho relativo, UTF-8 e divisão adequada de instruções. SQL comum pode usar `splitStatements: true`; funções/DO/PL/pgSQL exigem estratégia testada com o executor real. SQL externo deve sempre ter rollback explícito; ausência de reversão viável exige questionar a necessidade da alteração e registrar eventual exceção aceita, conforme [SQL](sql.md#8-rollback-obrigatório-e-exceções).

## 4. Exemplo de formato

**Exemplo didático; não adicionar ao master.** A tabela fictícia abaixo não é requisito de produto e pressupõe o schema `events`.

```yaml
databaseChangeLog:
  - changeSet:
      id: example-001
      author: scrapbook
      dbms: postgresql
      runInTransaction: true
      changes:
        - sqlFile:
            path: sql/001-create-example-markers.sql
            relativeToChangelogFile: true
            encoding: UTF-8
            splitStatements: true
            stripComments: false
      rollback:
        - sqlFile:
            path: rollback/001-create-example-markers.sql
            relativeToChangelogFile: true
            encoding: UTF-8
            splitStatements: false
            stripComments: false
```

SQL ilustrativo:

```sql
CREATE TABLE IF NOT EXISTS events.example_markers (
    id bigint GENERATED ALWAYS AS IDENTITY,
    name varchar(80) NOT NULL,
    scheduled_at timestamp without time zone NOT NULL,
    CONSTRAINT pk_example_markers_id PRIMARY KEY (id),
    CONSTRAINT ck_example_markers_name CHECK (btrim(name) <> '')
);

COMMENT ON COLUMN events.example_markers.scheduled_at IS
    'Horário de negócio em America/Sao_Paulo; exemplo sem requisito de produto.';
```

O exemplo pressupõe validação da estrutura existente e proveniência do objeto, conforme o guideline SQL; IF NOT EXISTS sozinho não comprova essas condições.

Rollback ilustrativo, somente para objeto criado por essa alteração:

```sql
DO $rollback$
BEGIN
    IF to_regclass('events.example_markers') IS NOT NULL THEN
        LOCK TABLE events.example_markers IN ACCESS EXCLUSIVE MODE;
        IF EXISTS (SELECT 1 FROM events.example_markers) THEN
            RAISE EXCEPTION 'Preservar os dados antes de remover example_markers.';
        END IF;
        DROP TABLE events.example_markers;
    END IF;
END;
$rollback$;
```

Ter rollback que remove a tabela não recupera os dados removidos. Neste exemplo, a remoção falha se houver dados. O banco gera o ID inteiro por identity e a PK garante sua unicidade; não gerar IDs por MAX(id) + 1.

## 5. Convenções e integridade

### Nomes e tipos

- `snake_case`, schema explícito, nomes sem aspas e campos comuns sem prefixos redundantes.
- FKs com nomes `<origem>_<destino>_id`; constraints e índices com nomes compostos `pk_`, `fk_`, `uq_`, `ck_`, `ix_`, conforme [nomenclatura SQL](sql.md#2-nomenclatura).
- Toda tabela tem PK simples `id bigint GENERATED ALWAYS AS IDENTITY`; FKs usam o mesmo tipo da chave referenciada. UUIDs técnicos, `sub`, job IDs e referências externas têm campos próprios conforme sua semântica.
- Datas e horários de negócio em `timestamp without time zone`, interpretados em `America/Sao_Paulo`, conforme o HLD.
- Instantes técnicos/externos devem ter representação e conversão explícitas no modelo, preservando a distinção em relação aos horários de negócio.
- Não depender do timezone da conexão/servidor para defaults ou comparação de prazo. Se usar horário atual local no SQL, tornar a conversão explícita.
- Tamanho em bytes e duração em unidade documentada, com limites coerentes entre API, domínio e banco.
- `jsonb` para metadados/contratos versionados quando justificado; não substituir relações centrais por JSON.

### Constraints e relacionamentos

Toda tabela de negócio tem PK. Campos obrigatórios usam `NOT NULL`; checks de faixa não substituem nulabilidade. Usar unicidade para concorrência persistente, FKs para relações aprovadas e índices conforme consultas.

O modelo deve impedir associações indevidas de arquivo/variante/sessão com registro ou evento diferente. Quando a relação atravessar módulos, registrar a opção por FK ou validação por contrato, considerando propriedade do schema e consistência; não criar acoplamento oculto.

Invariantes a detalhar no modelo:

- Até dois responsáveis e ausência de vínculo duplicado por evento.
- Convite de uso único, substituição e travamento sob concorrência.
- Uma tentativa lógica de upload por escopo/chave e conteúdo reconhecido.
- Confirmação integral do conjunto e referências válidas aos originais.
- Unicidade de consumo por consumidor/mensagem.
- Trabalho externo e notificação sem duplicação lógica por repetição.
- Auditoria e evidência de confirmação preservadas.

Limite entre linhas, como até dois responsáveis, não é resolvido por `CHECK` que consulta outras linhas. Definir protocolo transacional, slots/constraints ou alternativa explícita e testar disputa.

Definir `ON DELETE` pelo ciclo de vida. Não adicionar cascatas que removam lembranças confirmadas ou auditoria por conveniência. Triggers precisam de finalidade restrita e teste; não transferir autorização HTTP ou processamento de mídia para o banco.

## 6. Índices e consultas

Cada índice deve indicar consulta ou integridade atendida. Revisar índices existentes, ordem, seletividade e custo de escrita. PK/UNIQUE e FK têm funções diferentes; verificar suporte às colunas filhas e consultas reais.

Priorizar caminhos do produto: vínculos por conta/evento, sessão válida, galeria por evento/ordem/cursor, Outbox pendente, deduplicação, trabalhos e notificações pendentes. Não criar todos os índices possíveis sem consulta correspondente.

Avaliar planos com volume representativo. `EXPLAIN ANALYZE` executa a instrução; usar ambiente e comando adequados. Não adotar GIN para todo JSON, particionamento ou sharding nesta versão.

Em tabela povoada, considerar `CREATE INDEX CONCURRENTLY` quando o bloqueio convencional for inadequado, conforme versão adotada. Isolar a operação não transacional e documentar índice inválido/falha parcial; não misturar outros DDL no mesmo changeset. Tabela inicial vazia não exige esse custo por padrão.

## 7. Evolução, compatibilidade e locks

Preferir expansão → migração → contração para alterações incompatíveis:

1. Adicionar estrutura compatível.
2. Publicar código que conviva com o período de transição.
3. Transformar dados de forma limitada, observável e recuperável.
4. Validar integridade/completude e confirmar uso.
5. Remover estrutura antiga em entrega posterior.

O HLD não impõe coordenação automatizada das releases. Quando houver incompatibilidade, registrar ordem manual, janela de risco e recuperação; testar deploy parcial em homologação.

Para DDL em tabela povoada, analisar locks, varredura, reescrita, volume e duração. Definir limites de espera/execução. Constraints com validação posterior só quando suportadas, com etapa de conclusão rastreável; não deixar dados antigos indefinidamente sem validação.

Migração de dados não deve nascer escondida no startup da API nem criar transação única entre módulos.

## 8. Preconditions, drift e recuperação

Usar preconditions para premissas relevantes, com `HALT` em falha/erro essencial. Não usar `MARK_RAN` para aceitar divergência desconhecida.

Verificação prévia de dados não elimina corrida com escritores. O lock do Liquibase serializa migradores, não todas as operações da aplicação.

Após falha, conferir alvo, versão, histórico, locks e objetos antes de repetir. Só liberar lock órfão após confirmar ausência de executor ativo. Não editar tabelas de controle, apagar volumes ou recriar base compartilhada como recuperação habitual.

## 9. Entrega pelo GitHub Actions e Fargate

- Hlg aplica em homologação; main aplica em produção, automaticamente após merge.
- Pipeline autentica por OIDC, produz artefato/imagem nova versionada e executa tarefa Fargate de migração.
- A tarefa usa identidade própria, segredos privados, acesso ao RDS e nenhuma entrada de rede.
- Aplicação não executa DDL; migrador não reutiliza a identidade DML.
- Publicações e migrações precisam de serialização compatível com o ambiente; registrar execução e resultado.
- Infraestrutura é configurada manualmente, de forma reproduzível; nenhuma execução local direta substitui o fluxo controlado dos ambientes AWS.

Dev não tem banco remoto implantado. Desenvolvimento/validação usa PostgreSQL descartável local; homologação e produção são alvos somente dos pipelines previstos.

Documentar `validate`, `status`, geração/revisão do SQL e aplicação na versão escolhida. Validação sintática não prova execução, integridade ou impacto de locks. Comandos concretos e defaults privados serão definidos no README do repositório.

## 10. Validação própria e rollback

O Liquibase mantém validações próprias no pipeline, separadas das suítes manuais do repositório de testes. Fixar ferramentas e executar em banco descartável:

- Instalação vazia e upgrade a partir da versão anterior com dados.
- Reaplicação sem alteração pendente inesperada.
- Rejeição de dados inválidos, referências indevidas e duplicatas.
- Compatibilidade de schema com o mapeamento e versões em coexistência.
- Rollback/reaplicação quando reversíveis; correção futura/restauração quando necessário.
- Locks e concorrência nos casos afetados.

Registrar o que o validador efetivamente cobre e atualizar suas expectativas junto da evolução do schema. Ensaios reais de ambiente e cenários transversais são acionados manualmente.

Rollback do banco é **manual e somente pelo workflow Liquibase**. Selecionar ambiente, versão/changesets alvo, compatibilidade e risco de perda. Não embutir rollback automático no deploy.

Cada mudança declara reversão segura, destrutiva ou necessidade de correção/restauração. Não usar quantidade fixa de changesets como procedimento permanente. SQL inverso não recupera informação removida; backups RDS e versionamento S3 precisam de procedimento consistente com os registros confirmados.

Rollback é obrigatório. Se não for possível identificar uma reversão viável, preparar impacto e alternativas e questionar a necessidade da alteração ao responsável antes de executá-la. Só omitir o SQL de rollback mediante exceção explícita e registrada. Não usar arquivo vazio nem apagar objeto/linha que já existia antes da alteração; registrar o estado inicial e quais efeitos realmente foram aplicados.

## 11. Configuração e aceite

Credenciais em Secrets Manager ou configuração local privada ignorada, nunca argumentos/exemplos com senha. Restringir identidade por ambiente, schema e finalidade.

Execução deve ter `executionId`, versão, ambiente, resultado, duração e changesets; sem JDBC completo, dumps de dados ou segredos. Não presumir que a CLI já produza o JSON de observabilidade: adaptar e verificar no pipeline.

- [ ] ID, autor, caminho e ordem estáveis; histórico aplicado preservado.
- [ ] SQL/rollback, splitting, transação e recuperação revisados.
- [ ] Schemas por módulo e integridade entre evento/registro/arquivo coerentes.
- [ ] Datas locais de Brasília e conversões externas testadas.
- [ ] Constraints, índices, concorrência e locks atendem às regras.
- [ ] Instalação, upgrade e reaplicação têm evidência.
- [ ] Deploy independente e compatibilidade com versão anterior analisados.
- [ ] Rollback manual pelo workflow e perda de dados explicitados.
- [ ] Falta de rollback viável foi questionada; eventual exceção foi explicitamente aceita e registrada.
- [ ] Modelo físico e mapeamento EF acompanham a mudança.
