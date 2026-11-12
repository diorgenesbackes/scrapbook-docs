# Modelo físico proposto do banco

[Modelagem](DB-Modelagem.md) · [Diagramas](DB-Diagrama.md) · [Índice](README.md) · [SQL](../guidelines/sql.md)

Data inicial: **2026-11-11**. Especificação declarativa de referência. **Adendo de 2026-11-12:** scripts/rollbacks correspondentes estão em [scrapbook-liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md), com validação local em PostgreSQL 18.6. Não há seed de produto, mapping EF ou aplicação em RDS. Os detalhamentos P-01 a P-07 da [modelagem](DB-Modelagem.md#decisões-de-detalhamento-propostas) parametrizam esta base; recursos e integrações pendentes permanecem explícitos.

## 1. Convenções de leitura

- Cada uma das **33 tabelas** possui PK simples `id bigint GENERATED ALWAYS AS IDENTITY`, não nula, chamada `pk_<tabela>_id`. Unicidade do ID é por tabela. As chaves candidatas compostas não substituem essa PK.
- `Nulo? Não` significa `NOT NULL`. Campo sem default explícito deve ser fornecido pelo fluxo; não inferir defaults para state, bucket, datas ou segredos.
- Tipos `timestamptz` representam instantes técnicos normalizados em UTC; `timestamp without time zone` representa horário de negócio em `America/Sao_Paulo`. Usar precisão de microssegundos; conversão externa explícita.
- Todas as FKs usam `ON DELETE RESTRICT / ON UPDATE RESTRICT`. Nenhum CASCADE de negócio. FKs compostas opcionais usam `MATCH SIMPLE`: o ponteiro opcional nulo não invalida a coluna de escopo não nula. Quando o ponteiro existe, todas as partes devem existir e corresponder.
- UNIQUE sem filtro é chave candidata com índice B-tree implícito. UNIQUE parcial é índice único, não alvo de FK. Índices `ix_*` são B-tree propostos, ordem ascendente, salvo revisão demonstrada por plano real.
- UNIQUE de campos opcionais mantém NULLs distintos; não usar `NULLS NOT DISTINCT` por padrão. Não duplicar índices implícitos de PK/UNIQUE.
- CHECK valida somente a própria linha. Regras entre linhas estão na seção 7; autorização e prova de conteúdo S3/Cognito permanecem na aplicação/adaptadores. Não usar CHECK ou predicado de índice dependente de `now()`.
- `bytea` de hash/HMAC exige comprimento exato de 32 bytes. Ciphertexts são envelopes autenticados de tamanho variável, com chave identificável; não são mídia no banco.
- Buckets: 3..63 caracteres e regras S3; keys: 1..1024 bytes UTF-8 incluindo prefixo. IDs opacos de provedores são `text`, sem limite de negócio inventado; adaptadores impõem limites do contrato antes de persistir. Índices desses IDs usam digest de tamanho fixo e comparação do valor integral para detectar colisão.

[Constraints PostgreSQL](https://www.postgresql.org/docs/18/ddl-constraints.html), [chaves S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html), [nomes de buckets S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html).

### Metadados comuns

| Coluna | Tipo | Nulo? | Regra |
| --- | --- | --- | --- |
| id | bigint | Não | Identity ALWAYS; PK nomeada; não alterar/reutilizar. |
| created_at | timestamptz | Não | Instante técnico do servidor; default conceitual instante de criação. |
| updated_at | timestamptz | Não, em tabelas mutáveis | Default conceitual created_at; atualizar só em mudança efetiva. |
| version | bigint | Não, em tabelas mutáveis | Default 0; `ck_<tabela>_version` garante >=0; aplicação compara a versão esperada; trigger incrementa e atualiza updated_at em mudança efetiva. |

Tabelas imutáveis: `events.event_members`, `records.consent_notices` e as quatro `audit_entries`. Imutabilidade é política de escrita; expurgo autorizado não é edição de conteúdo. Todas as demais recebem os quatro metadados. Se existirem `lease_token` e `lease_until`, `ck_<tabela>_lease` (ou CHECK de estado indicado) exige ambos nulos ou ambos preenchidos; a comparação do token na conclusão é condicional, não um CHECK de concorrência.

### Inventário por proprietário

| Schema / módulo | Tabelas de domínio e operação | Tabelas técnicas locais |
| --- | --- | --- |
| `identity` | `accounts`, `sessions`, `oidc_requests`, `account_operations` | `outbox_messages`, `processed_messages`, `audit_entries` |
| `events` | `events`, `event_members`, `invitations`, `covers`, `opening_dispatches` | `outbox_messages`, `processed_messages`, `audit_entries` |
| `records` | `consent_notices`, `records`, `idempotent_operations`, `upload_sessions`, `files`, `storage_objects`, `file_variants`, `processing_jobs`, `processing_attempts`, `cleanup_jobs` | `outbox_messages`, `processed_messages`, `audit_entries` |
| `notifications` | `notifications`, `notification_attempts` | `outbox_messages`, `processed_messages`, `audit_entries` |

Total: **21 tabelas de domínio/operação + 12 instâncias técnicas**, todas com ownership e transação locais. `DATABASECHANGELOG` e `DATABASECHANGELOGLOCK`, tabelas internas do Liquibase, seguem o contrato da ferramenta e não são entidades da aplicação.
## 2. Contas e Acesso — `identity`

### `identity.accounts`

Conta local vinculada ao Cognito; fonte local de perfil confirmado e bloqueio/revogação.

Requisitos: FR-001, FR-002. PK: `pk_accounts_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `subject` | `varchar(255)` | Sim | sub OIDC, case-sensitive; obrigatório salvo conta anonimizada. Não é a PK. |
| `name` | `varchar(120)` | Sim | Nome de exibição; limite proposto P-01; obrigatório no estado ativo. |
| `email` | `varchar(320)` | Sim | E-mail confirmado atual; limite proposto P-01. |
| `normalized_email` | `varchar(320)` | Sim | Chave canônica; normalização deve ser compatível com o User Pool. |
| `email_verified` | `boolean` | Não | Default false; só verdadeiro após comprovação Cognito. |
| `state` | `varchar(32)` | Não | active, revocation_pending, disabled ou anonymized. |
| `security_version` | `bigint` | Não | Default 0; gera invalidação de todas as sessões anteriores. |
| `synchronized_at` | `timestamptz` | Sim | Última reconciliação de perfil com o provedor. |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_accounts_state` | state pertence ao conjunto documentado; security_version e version não negativos. |
| `ck_accounts_profile` | active/revocation_pending/disabled exigem subject, name, email e normalized_email não vazios; anonymized permite limpeza desses campos e email_verified=false. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_accounts_subject` | `subject` | Sim | Unicidade/invariante. |
| `uq_accounts_normalized_email` | `normalized_email` | Sim | Unicidade/invariante. |

**Regras complementares**

- Cognito é dono de senha e verificação. A conta local surge após identidade confirmada; não há tabela de senhas.
- Anonymized é suporte a procedimento fora da interface, sujeito à revisão de privacidade; mantém a PK para vínculos históricos.

### `identity.sessions`

Sessão BFF compartilhada entre réplicas, com segredo de acesso protegido e tokens criptografados.

Requisitos: FR-001, FR-002. PK: `pk_sessions_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `sessions_accounts_id` | `bigint` | Não | Conta titular. |
| `token_hash` | `bytea` | Não | 32 bytes; HMAC do segredo aleatório do cookie; nunca o valor bruto. |
| `hash_key_id` | `varchar(64)` | Não | Versão da chave de HMAC, proposta P-04. |
| `access_token_ciphertext` | `bytea` | Sim | Envelope autenticado; obrigatório enquanto active/refreshing. |
| `refresh_token_ciphertext` | `bytea` | Sim | Envelope autenticado; presença conforme concessão Cognito. |
| `encryption_key_id` | `varchar(64)` | Sim | Identificador interno da chave; obrigatório se existir ciphertext. |
| `account_security_version` | `bigint` | Não | Versão da conta capturada na emissão. |
| `state` | `varchar(32)` | Não | active, refreshing, revoked ou expired. |
| `expires_at` | `timestamptz` | Não | Prazo absoluto, no máximo criação + 7 dias; sem renovação deslizante implícita. |
| `access_expires_at` | `timestamptz` | Não | exp do token validado, normalizado para UTC. |
| `refresh_expires_at` | `timestamptz` | Sim | Prazo efetivo da concessão de refresh; não confundir com a sessão local. |
| `revoked_at` | `timestamptz` | Sim | Momento de revogação local. |
| `revocation_reason` | `varchar(80)` | Sim | Código de catálogo, sem texto do provedor. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Expiração técnica da posse; preenchido junto com lease_token. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_sessions_accounts_id` | `sessions_accounts_id` → `identity.accounts` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_sessions_lifetime` | expires_at > created_at e expires_at <= created_at + 7 dias; renovação não amplia esse teto. |
| `ck_sessions_state` | state válido; revoked exige revoked_at; estado ativo não dispensa comparação do prazo. |
| `ck_sessions_secrets` | token_hash tem 32 bytes; ciphertext não vazio; par lease_token/lease_until nulo ou preenchido; versão de segurança não negativa. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_sessions_token_hash` | `hash_key_id`, `token_hash` | Sim | Unicidade/invariante. |
| `ix_sessions_accounts_state` | `sessions_accounts_id`, `state`, `id` | Não | Revogar/listar sessões da conta. |
| `ix_sessions_expires_at` | `expires_at`, `id` | Não | Limpeza e localização de expiradas. |

**Regras complementares**

- Validar sessão, conta ativa, prazo e security_version a cada acesso.
- Lease de refresh impede duas réplicas de renovar simultaneamente; efeitos externos não entram na transação SQL.

### `identity.oidc_requests`

Estado de uma tentativa de login/callback, nonce e PKCE do backend.

Requisitos: FR-001. PK: `pk_oidc_requests_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `state_hash` | `bytea` | Não | 32 bytes; hash/HMAC do state enviado ao navegador. |
| `nonce_hash` | `bytea` | Não | 32 bytes; validação do nonce do ID token. |
| `browser_hash` | `bytea` | Não | HMAC de 32 bytes do segredo de vínculo ao navegador iniciador; cookie temporário protegido, distinto de state/nonce. |
| `hash_key_id` | `varchar(64)` | Não | Versão da chave para os hashes. |
| `verifier_ciphertext` | `bytea` | Sim | PKCE verifier protegido; apagar após conclusão/expiração. |
| `encryption_key_id` | `varchar(64)` | Sim | Chave do envelope, se houver verifier. |
| `return_path` | `varchar(2048)` | Não | Somente caminho relativo validado; limite técnico proposto P-04. |
| `state` | `varchar(32)` | Não | pending, processing, completed, failed ou expired. |
| `expires_at` | `timestamptz` | Não | TTL proposto de 10 minutos, P-04. |
| `completed_at` | `timestamptz` | Sim | Conclusão ou consumo terminal. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Expiração técnica da posse; preenchido junto com lease_token. |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_oidc_requests_state` | state válido; completed exige completed_at. |
| `ck_oidc_requests_secrets` | state_hash, nonce_hash e browser_hash têm 32 bytes; return_path é relativo; expires_at > created_at; lease em par. Allowlist de rotas é validação de aplicação. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_oidc_requests_state_hash` | `hash_key_id`, `state_hash` | Sim | Unicidade/invariante. |
| `ix_oidc_requests_expires_at` | `expires_at`, `id` | Não | Expiração e limpeza. |

**Regras complementares**

- Não persistir authorization code em texto, senha ou resposta integral do provedor.
- Código consumido com resultado externo desconhecido não deve ser trocado novamente cegamente; novo login pode ser necessário.
- Callback exige state, nonce e vínculo ao navegador iniciador; existência de state no banco isoladamente não impede login CSRF.

### `identity.account_operations`

Coordenação/reconciliação de alterações locais e Cognito, sem persistir senhas.

Requisitos: FR-002. PK: `pk_account_operations_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `account_operations_accounts_id` | `bigint` | Não | Conta afetada. |
| `request_key` | `uuid` | Não | Identidade da operação; não é autorização. |
| `kind` | `varchar(32)` | Não | email_change, password_change, password_recovery, session_revocation ou profile_sync. |
| `state` | `varchar(32)` | Não | pending, provider_pending, verification_pending, reconciling, completed ou failed. |
| `target_name` | `varchar(120)` | Sim | Novo nome quando aplicável; P-01. |
| `target_email_ciphertext` | `bytea` | Sim | Novo e-mail protegido até confirmação; não substituir o atual antes disso. |
| `encryption_key_id` | `varchar(64)` | Sim | Chave do envelope. |
| `provider_reference` | `text` | Sim | Referência opaca externa, quando fornecida; sem payload ou código secreto. |
| `applied_security_version` | `bigint` | Sim | Versão local bloqueada/revogada por esta operação. |
| `attempt_count` | `integer` | Não | Default 0; tentativas de reconciliação. |
| `next_attempt_at` | `timestamptz` | Sim | Nova tentativa elegível. |
| `completed_at` | `timestamptz` | Sim | Conclusão terminal. |
| `last_error_code` | `varchar(80)` | Sim | Erro classificado; sem senha, código Cognito ou mensagem livre. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Expiração técnica da posse; preenchido junto com lease_token. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_account_operations_accounts_id` | `account_operations_accounts_id` → `identity.accounts` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_account_operations_state` | kind/state válidos; attempt_count não negativo; completed exige completed_at; lease em par. |
| `ck_account_operations_encryption` | target_email_ciphertext exige encryption_key_id; nenhuma coluna admite senha/código de recuperação. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_account_operations_request` | `account_operations_accounts_id`, `kind`, `request_key` | Sim | Unicidade/invariante. |
| `ix_account_operations_pending` | `next_attempt_at`, `id` | Não | Parcial: state em pending/provider_pending/reconciling. Reconciliador local. |

**Regras complementares**

- Troca/recuperação de senha exige revogação das sessões; o sinal confiável do Cognito é uma dependência de integração, não resolvida apenas pela tabela.
- Recuperação usa código Cognito de uma hora; não criar tabela de links de 30 minutos.

## 3. Eventos e Convites — `events`

### `events.events`

Configuração do evento, janela de envio e política de abertura.

Requisitos: FR-003, FR-005, FR-006, FR-016, FR-017, FR-018. PK: `pk_events_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `events_creator_accounts_id` | `bigint` | Não | Conta criadora; metadado, sem privilégios superiores ao segundo responsável. |
| `events_covers_id` | `bigint` | Sim | Capa personalizada atual; NULL usa a capa padrão. |
| `code` | `varchar(43)` | Não | Código aleatório de 256 bits em base64url, proposta P-03; link público/QR. |
| `name` | `varchar(120)` | Não | Nome público do evento; limite proposto P-01. |
| `welcome_message` | `varchar(2000)` | Sim | NULL usa mensagem padrão; limite proposto P-01. |
| `starts_at` | `timestamp without time zone` | Não | Data/hora civil em Brasília. |
| `submission_opens_at` | `timestamp without time zone` | Não | Snapshot derivado: starts_at menos 2 horas. |
| `submission_closes_at` | `timestamp without time zone` | Não | Snapshot derivado: meio-dia do dia seguinte. |
| `completion_deadline_at` | `timestamp without time zone` | Não | Snapshot derivado: 13h do dia seguinte. |
| `opening_mode` | `varchar(16)` | Não | immediate ou scheduled; padrão funcional scheduled. |
| `opens_at` | `timestamp without time zone` | Sim | Meia-noite da data futura; obrigatório em scheduled, NULL em immediate. |
| `access_state` | `varchar(24)` | Não | flexible, scheduled_locked ou immediate_once; congelamento efetivo também considera o relógio. |
| `access_frozen_at` | `timestamp without time zone` | Sim | Marco de congelamento; mantém-se mesmo se o evento sem registros mudar de data. |
| `access_version` | `bigint` | Não | Default 1; altera somente com mudança de configuração de abertura. |
| `locked_at` | `timestamptz` | Sim | Travamento estrutural aplicado pelo consumidor, após primeira confirmação. |
| `first_record_id` | `bigint` | Sim | Referência lógica ao registro que motivou o travamento; sem FK cross-module. |
| `first_record_confirmed_at` | `timestamp without time zone` | Sim | Instante de negócio informado pela mensagem; base da latência de travamento. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_events_creator_accounts_id` | `events_creator_accounts_id` → `identity.accounts` (`id`) |
| `fk_events_covers_event_id` | `id`, `events_covers_id` → `events.covers` (`covers_events_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_events_name` | name não vazio; code com 43 caracteres base64url. |
| `ck_events_window` | submission_opens_at = starts_at - 2h; fechamento ao meio-dia seguinte; tolerância às 13h; ordem estrita entre os marcos. |
| `ck_events_opening` | opening_mode válido; scheduled exige opens_at à meia-noite e posterior a submission_closes_at; immediate exige opens_at nulo; access_state válido, scheduled_locked implica scheduled, immediate_once implica immediate; access_state não flexible exige access_frozen_at; access_version >= 1. A comparação com o dia atual é feita na alteração, não em CHECK temporal. |
| `ck_events_locking` | locked_at, first_record_id e first_record_confirmed_at preenchidos em conjunto; first_record_id positivo. Não exigir FK para esse histórico lógico. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_events_code` | `code` | Sim | Unicidade/invariante. |
| `ix_events_creator` | `events_creator_accounts_id`, `id` | Não | Integridade e rastreio da criação; listagem por membros usa event_members. |
| `ix_events_opening` | `opens_at`, `id` | Não | Parcial: opening_mode=scheduled. Agendador de abertura. |

**Regras complementares**

- Criação grava evento e membro slot 1 na mesma transação Events.
- Snapshots de janela devem corresponder a starts_at; mudança estrutural revalida a política de abertura antes de atualizar a janela.
- Não armazenar contagem de registros/remetentes para a página bloqueada.

### `events.event_members`

Vínculo de um ou dois responsáveis com permissões iguais.

Requisitos: FR-003, FR-004. PK: `pk_event_members_id`. Metadados: padrão imutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `event_members_events_id` | `bigint` | Não | Evento. |
| `event_members_accounts_id` | `bigint` | Não | Conta responsável. |
| `slot` | `smallint` | Não | 1 ou 2; capacidade persistente, não papel de autorização. |
| `joined_at` | `timestamptz` | Não | Adesão confirmada. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_event_members_events_id` | `event_members_events_id` → `events.events` (`id`) |
| `fk_event_members_accounts_id` | `event_members_accounts_id` → `identity.accounts` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_event_members_slot` | slot pertence a {1,2}. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_event_members_event_slot` | `event_members_events_id`, `slot` | Sim | Unicidade/invariante. |
| `uq_event_members_event_account` | `event_members_events_id`, `event_members_accounts_id` | Sim | Unicidade/invariante. |
| `uq_event_members_event_scope` | `event_members_events_id`, `id` | Sim | Unicidade/invariante. |
| `ix_event_members_account_events` | `event_members_accounts_id`, `event_members_events_id` | Não | Listar os eventos da conta. |

**Regras complementares**

- O primeiro membro é criado junto do evento; não há fluxo de remoção/substituição de responsáveis nesta versão.
- Unicidade de slot impede terceiro responsável sob concorrência; conta não pode ocupar os dois slots.

### `events.invitations`

Convite de uso único; novo convite substitui o anterior.

Requisitos: FR-004, FR-021. PK: `pk_invitations_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `invitations_events_id` | `bigint` | Não | Evento convidado. |
| `invitations_issuer_event_members_id` | `bigint` | Não | Membro emissor do mesmo evento. |
| `invitations_recipient_accounts_id` | `bigint` | Sim | Conta que aceitou; preenchida somente no aceite. |
| `request_key` | `uuid` | Não | Repetição da mesma geração de convite sem criar outro token. |
| `token_hash` | `bytea` | Não | 32 bytes; validação do segredo do link/QR. |
| `hash_key_id` | `varchar(64)` | Não | Versão da chave HMAC. |
| `token_ciphertext` | `bytea` | Sim | Segredo protegido para reexibição autorizada e envio por Notificações; apagar ao encerrar. |
| `encryption_key_id` | `varchar(64)` | Sim | Chave do envelope. |
| `state` | `varchar(24)` | Não | pending, accepted, superseded, invalidated ou expired. |
| `expires_at` | `timestamptz` | Não | Criação + 24 horas; validar no pedido, sem depender de job. |
| `accepted_at` | `timestamptz` | Sim | Aceite efetivo. |
| `invalidated_at` | `timestamptz` | Sim | Substituição/travamento/expiração materializada. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_invitations_events_id` | `invitations_events_id` → `events.events` (`id`) |
| `fk_invitations_event_members_event_id` | `invitations_events_id`, `invitations_issuer_event_members_id` → `events.event_members` (`event_members_events_id`, `id`) |
| `fk_invitations_recipient_accounts_id` | `invitations_recipient_accounts_id` → `identity.accounts` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_invitations_state` | state válido; accepted exige accepted_at e conta receptora; demais estados não têm conta receptora. |
| `ck_invitations_lifetime` | expires_at = created_at + 24h; accepted_at, se presente, anterior a expires_at. |
| `ck_invitations_secrets` | token_hash de 32 bytes; ciphertext exige chave; pending exige segredo protegido disponível. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_invitations_request` | `invitations_events_id`, `request_key` | Sim | Unicidade/invariante. |
| `uq_invitations_token_hash` | `hash_key_id`, `token_hash` | Sim | Unicidade/invariante. |
| `uq_invitations_active` | `invitations_events_id` | Sim | Parcial: state=pending. Unicidade/invariante. |
| `uq_invitations_event_scope` | `invitations_events_id`, `id` | Sim | Unicidade/invariante. |
| `ix_invitations_expires_at` | `expires_at`, `id` | Não | Expirar convites e apagar ciphertext. |
| `ix_invitations_issuer` | `invitations_issuer_event_members_id` | Não | Suporte à FK de emissor. |
| `ix_invitations_recipient` | `invitations_recipient_accounts_id` | Não | Parcial: invitations_recipient_accounts_id preenchido. Suporte à conta que aceitou. |

**Regras complementares**

- E-mail de destino é dado da Notificação de convite, não vincula automaticamente a uma conta.
- Só pending, não expirado e evento não travado pode gerar novo vínculo.

### `events.covers`

Arquivo de capa sob propriedade de Events; original/temporário de preparação não são lembranças privadas.

Requisitos: FR-003, FR-005. PK: `pk_covers_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `covers_events_id` | `bigint` | Não | Evento titular. |
| `state` | `varchar(24)` | Não | pending, validating, ready, superseded, rejected, deleting ou deleted. |
| `bucket` | `varchar(63)` | Não | Bucket privado de capas. |
| `object_key` | `text` | Não | Chave opaca; entre 1 e 1024 bytes UTF-8; sem nome pessoal. |
| `version_id` | `text` | Sim | Versão S3 opaca, obrigatória após persistência ready. |
| `temporary_key` | `text` | Sim | Chave temporária enquanto a capa é recebida/preparada. |
| `temporary_version_id` | `text` | Sim | Versão temporária fixada para validação. |
| `declared_size` | `integer` | Não | Bytes; proposta P-02 de até 20.000.000 para capa. |
| `actual_size` | `bigint` | Sim | Bytes aferidos, inclusive tamanho inválido para diagnóstico da rejeição; limite do tipo é exigido antes de ready/valid. |
| `media_type` | `varchar(128)` | Sim | Tipo efetivo validado, não apenas o declarado. |
| `width` | `integer` | Sim | Largura de exibição validada. |
| `height` | `integer` | Sim | Altura de exibição validada. |
| `sha256` | `bytea` | Sim | 32 bytes, checksum do objeto validado. |
| `received_at` | `timestamptz` | Sim | Recebimento do arquivo no armazenamento. |
| `ready_at` | `timestamptz` | Sim | Pronto para publicar via CloudFront. |
| `cleanup_after` | `timestamptz` | Sim | Limpeza de capa rejeitada/sobrescrita, proposta P-05. |
| `deleted_at` | `timestamptz` | Sim | Remoção do objeto confirmada. |
| `attempt_count` | `integer` | Não | Default 0; tentativas de preparação/limpeza. |
| `last_error_code` | `varchar(80)` | Sim | Código sanitizado. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Expiração técnica da posse; preenchido junto com lease_token. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_covers_events_id` | `covers_events_id` → `events.events` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_covers_state` | state válido; ready exige version_id, actual_size, tipo, dimensões e ready_at; deleted exige deleted_at. |
| `ck_covers_size` | declared_size em 1..20.000.000; actual_size positivo quando presente e <=20.000.000 em ready; dimensões positivas; sha256 com 32 bytes quando presente. |
| `ck_covers_storage` | Bucket válido de 3..63 caracteres; chaves presentes entre 1 e 1024 bytes; lease em par. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_covers_event_scope` | `covers_events_id`, `id` | Sim | Unicidade/invariante. |
| `uq_covers_object_key` | `bucket`, `object_key` | Sim | Unicidade/invariante. |
| `ix_covers_cleanup` | `cleanup_after`, `id` | Não | Parcial: state em superseded/rejected/deleting. Buscar capas elegíveis à limpeza. |

**Regras complementares**

- Atribuição de capa atual revalida locked_at e estado ready na transação Events.
- Não apagar objeto ainda selecionado por events.events_covers_id; estado deleting impede nova seleção.

### `events.opening_dispatches`

Obrigação de aviso de abertura por responsável e versão da política; acompanha conclusão eventual.

Requisitos: FR-017, FR-021. PK: `pk_opening_dispatches_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `opening_dispatches_events_id` | `bigint` | Não | Evento elegível. |
| `opening_dispatches_event_members_id` | `bigint` | Não | Destinatário membro do mesmo evento. |
| `access_version` | `bigint` | Não | Versão da configuração de abertura. |
| `due_at` | `timestamp without time zone` | Não | Meia-noite de abertura em Brasília. |
| `state` | `varchar(24)` | Não | pending, requested, dispatched ou cancelled. |
| `requested_at` | `timestamptz` | Sim | Intenção enviada à Outbox local. |
| `dispatched_at` | `timestamptz` | Sim | Confirmação de encaminhamento recebida de Notificações. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_opening_dispatches_events_id` | `opening_dispatches_events_id` → `events.events` (`id`) |
| `fk_opening_dispatches_event_members_event_id` | `opening_dispatches_events_id`, `opening_dispatches_event_members_id` → `events.event_members` (`event_members_events_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_opening_dispatches_state` | state válido; access_version >= 1; requested exige requested_at; dispatched exige requested_at e dispatched_at. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_opening_dispatches_recipient_version` | `opening_dispatches_events_id`, `opening_dispatches_event_members_id`, `access_version` | Sim | Unicidade/invariante. |
| `uq_opening_dispatches_event_scope` | `opening_dispatches_events_id`, `id` | Sim | Unicidade/invariante. |
| `ix_opening_dispatches_member` | `opening_dispatches_event_members_id` | Não | Suporte ao vínculo de destinatário. |
| `ix_opening_dispatches_due` | `due_at`, `id` | Não | Parcial: state em pending/requested. Avisos ainda não concluídos. |

**Regras complementares**

- Unicidade mantém no máximo uma obrigação por versão/membro; o e-mail não libera o acesso.
- Notificações comunica conclusão por mensagem; não existe commit simultâneo entre os módulos.

## 4. Registros e Mídias — `records`

### `records.consent_notices`

Versões imutáveis do aviso de acesso aceito pelo convidado; conteúdo controlado pelo produto.

Requisitos: FR-007. PK: `pk_consent_notices_id`. Metadados: padrão imutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `code` | `varchar(32)` | Não | Versão funcional única do aviso; proposta P-01. |
| `body` | `text` | Não | Texto em português; fonte controlada, não campo livre do convidado. |
| `body_hash` | `bytea` | Não | 32 bytes, SHA-256 da versão publicada. |
| `effective_at` | `timestamptz` | Não | Início de vigência; seleção pela aplicação. |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_consent_notices_content` | code/body não vazios; body_hash tem 32 bytes. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_consent_notices_code` | `code` | Sim | Unicidade/invariante. |
| `ix_consent_notices_effective` | `effective_at`, `id` | Não | Selecionar versão vigente. |

**Regras complementares**

- Não editar versão já referenciada; publicar nova linha.
- Concordância e nome não provam identidade nem conformidade legal isoladamente.

### `records.records`

Envio lógico indivisível: depoimento ou momento; criado provisoriamente e confirmado uma única vez.

Requisitos: FR-007, FR-008, FR-009, FR-011, FR-012, FR-014, FR-015, FR-016, FR-019, FR-020. PK: `pk_records_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `records_events_id` | `bigint` | Não | Evento titular, autorizado por contrato Events. |
| `records_consent_notices_id` | `bigint` | Não | Aviso aceito neste envio. |
| `category` | `varchar(16)` | Não | testimonial ou moment. |
| `format` | `varchar(8)` | Não | photo, video, audio ou text. |
| `sender_name` | `varchar(120)` | Sim | Nome/apelido congelado no envio; obrigatório salvo rejeitado já saneado; P-01. |
| `message` | `varchar(5000)` | Sim | Somente depoimento em texto; obrigatório/não vazio nesse formato enquanto não saneado. |
| `caption` | `varchar(500)` | Sim | Legenda opcional de mídia, comum ao conjunto. |
| `expected_file_count` | `smallint` | Não | 0 para texto; 1 para foto de depoimento/vídeo/áudio; 1..10 para fotos de momento. |
| `consented_at` | `timestamp without time zone` | Não | Concordância recebida pelo servidor, ligada ao nome e aviso deste registro. |
| `submitted_at` | `timestamp without time zone` | Não | Início autorizado recebido pelo backend, após revisão. |
| `received_at` | `timestamp without time zone` | Sim | Recebimento integral do conjunto ou texto, por evidência confiável. |
| `confirmed_at` | `timestamp without time zone` | Sim | Fato imutável de confirmação, distinto do término da conversão. |
| `state` | `varchar(24)` | Não | receiving, validating, confirmed, processing, available, rejected ou processing_failed. |
| `rejection_code` | `varchar(80)` | Sim | Motivo classificado de rejeição de recebimento. |
| `redacted_at` | `timestamptz` | Sim | Limpeza de conteúdo/nome de envio não confirmado; proposta P-05. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_records_events_id` | `records_events_id` → `events.events` (`id`) |
| `fk_records_consent_notices_id` | `records_consent_notices_id` → `records.consent_notices` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_records_kind` | category/format válidos; moment admite somente photo/video; expected_file_count conforme formato/categoria. |
| `ck_records_content` | Enquanto redacted_at for nulo: texto exige message não vazia, caption nula e zero arquivos; mídia exige message nula; sender_name não vazio. Redação só é admitida na condição de ck_records_redaction. |
| `ck_records_confirmation` | received_at e confirmed_at preenchidos em conjunto nos estados confirmed/processing/available/processing_failed; receiving/validating/rejected mantêm confirmed_at nulo; confirmed_at >= received_at >= submitted_at. |
| `ck_records_redaction` | redacted_at somente em rejeitado não confirmado; message/caption/sender_name podem ser limpos nesse caso. |
| `ck_records_state` | state válido; rejeitado exige rejection_code; não usar CHECK com consultas em outras tabelas. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_records_event_scope` | `records_events_id`, `id` | Sim | Unicidade/invariante. |
| `ix_records_gallery` | `records_events_id`, `submitted_at`, `id` | Não | Parcial: confirmed_at preenchido. Galeria Tudo com cursor. |
| `ix_records_gallery_category` | `records_events_id`, `category`, `submitted_at`, `id` | Não | Parcial: confirmed_at preenchido. Filtros de categoria com cursor. |
| `ix_records_notice` | `records_consent_notices_id` | Não | Suporte à versão do aviso. |

**Regras complementares**

- Não há FK para uma identidade de convidado; cada envio guarda seu próprio nome e consentimento.
- received_at/confirmed_at não são regravados para simular novo envio após retry.
- Não há edição/exclusão pela interface nem tabela de busca por remetente.

### `records.idempotent_operations`

Identidade da tentativa lógica e autorização separada da chave idempotente.

Requisitos: FR-006, FR-014, FR-015. PK: `pk_idempotent_operations_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `idempotent_operations_events_id` | `bigint` | Não | Escopo do evento. |
| `idempotent_operations_records_id` | `bigint` | Não | Registro provisório ou confirmado desta tentativa. |
| `operation` | `varchar(40)` | Não | Nesta versão: contribution.submit. |
| `key` | `uuid` | Não | Chave idempotente escolhida antes da primeira solicitação; nunca credencial. |
| `payload_hash` | `bytea` | Sim | 32 bytes; HMAC do request/manifesto canônico com chave versionada, não comprova os bytes S3. |
| `authorization_hash` | `bytea` | Sim | 32 bytes; HMAC de segredo separado, conhecido pelo navegador antes do primeiro envio; P-03. |
| `hash_key_id` | `varchar(64)` | Sim | Chave dos HMACs enquanto recuperável. |
| `authorization_expires_at` | `timestamptz` | Não | Prazo de replay/status, proposta P-04; não amplia o prazo de upload. |
| `state` | `varchar(24)` | Não | in_progress, confirmed, rejected ou expired. |
| `completed_at` | `timestamptz` | Sim | Resultado lógico terminal. |
| `result_code` | `varchar(80)` | Sim | Código seguro de resultado; sem texto, mídia, cookies ou URLs. |
| `retired_at` | `timestamptz` | Sim | Retirada dos hashes de replay ao fim da retenção; mantém tombstone de unicidade. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_idempotent_operations_events_id` | `idempotent_operations_events_id` → `events.events` (`id`) |
| `fk_idempotent_operations_records_event_id` | `idempotent_operations_events_id`, `idempotent_operations_records_id` → `records.records` (`records_events_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_idempotent_operations_state` | state válido; terminal exige completed_at; operação em progresso não pode ser aposentada. |
| `ck_idempotent_operations_hashes` | Enquanto retired_at nulo, ambos os hashes e hash_key_id são obrigatórios; hashes presentes têm 32 bytes. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_idempotent_operations_scope_key` | `idempotent_operations_events_id`, `operation`, `key` | Sim | Unicidade/invariante. |
| `uq_idempotent_operations_record` | `idempotent_operations_records_id` | Sim | Unicidade/invariante. |
| `uq_idempotent_operations_event_scope` | `idempotent_operations_events_id`, `id` | Sim | Unicidade/invariante. |
| `ix_idempotent_operations_retire` | `authorization_expires_at`, `id` | Não | Parcial: retired_at nulo. Retirar dados de replay expirados. |

**Regras complementares**

- Uma resposta perdida pode ser recuperada pela mesma chave e pelo segredo separado; a chave isolada não autoriza leitura.
- A unicidade do vínculo a record dura enquanto existir o envio; segredo/fingerprint podem ser removidos após o prazo.
- A rejeição da operação não autoriza reutilizar a chave para outro payload.

### `records.upload_sessions`

Plano autorizado de recebimento, snapshot de janela e ligação à tentativa lógica.

Requisitos: FR-006, FR-014, FR-015. PK: `pk_upload_sessions_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `upload_sessions_events_id` | `bigint` | Não | Escopo redundante para garantir igualdade entre record e operação. |
| `upload_sessions_records_id` | `bigint` | Não | Registro desta sessão; relação 1:1. |
| `upload_sessions_idempotent_operations_id` | `bigint` | Não | Operação idempotente correspondente; relação 1:1. |
| `state` | `varchar(24)` | Não | authorized, receiving, validating, confirmed, rejected ou expired. |
| `window_opens_at` | `timestamp without time zone` | Não | Janela capturada na admissão. |
| `window_closes_at` | `timestamp without time zone` | Não | Fechamento capturado na admissão. |
| `completion_deadline_at` | `timestamp without time zone` | Não | 13h do dia seguinte capturado na admissão. |
| `started_at` | `timestamp without time zone` | Não | Solicitação recebida pelo servidor antes do fechamento. |
| `sealed_at` | `timestamptz` | Sim | Manifesto/versões recebidas fixados para validação; não é confirmação. |
| `terminal_at` | `timestamptz` | Sim | Estado terminal de recebimento. |
| `last_error_code` | `varchar(80)` | Sim | Motivo seguro de falha. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_upload_sessions_events_id` | `upload_sessions_events_id` → `events.events` (`id`) |
| `fk_upload_sessions_records_event_id` | `upload_sessions_events_id`, `upload_sessions_records_id` → `records.records` (`records_events_id`, `id`) |
| `fk_upload_sessions_idempotent_operations_event_id` | `upload_sessions_events_id`, `upload_sessions_idempotent_operations_id` → `records.idempotent_operations` (`idempotent_operations_events_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_upload_sessions_window` | window_opens_at <= started_at < window_closes_at < completion_deadline_at; os três marcos pertencem à janela capturada. |
| `ck_upload_sessions_state` | state válido; confirmed/rejected/expired exigem terminal_at. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_upload_sessions_record` | `upload_sessions_records_id` | Sim | Unicidade/invariante. |
| `uq_upload_sessions_operation` | `upload_sessions_idempotent_operations_id` | Sim | Unicidade/invariante. |
| `ix_upload_sessions_event` | `upload_sessions_events_id`, `id` | Não | Localizar autorizações por evento. |
| `ix_upload_sessions_deadline` | `completion_deadline_at`, `id` | Não | Parcial: state em authorized/receiving/validating. Reconciliar recebimentos pendentes. |

**Regras complementares**

- Replays autorizados de uma confirmação anterior não exigem janela de novo envio aberta.
- Arquivo integralmente recebido no prazo pode ser validado/confirmado depois das 13h.
- Expiração depende de reconciliação com evidências de S3; não basta não ter recebido uma notificação até as 13h.

### `records.files`

Arquivo esperado e validado de um registro; posição estável para o conjunto.

Requisitos: FR-009, FR-010, FR-011, FR-014, FR-019, FR-020. PK: `pk_files_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `files_records_id` | `bigint` | Não | Registro pai; evento herdado por esse vínculo. |
| `position` | `smallint` | Não | Ordem no conjunto: 1..10, única por registro. |
| `client_key` | `uuid` | Não | Identidade do item no manifesto; mantém correspondência no retry. |
| `kind` | `varchar(8)` | Não | photo, video ou audio; deve corresponder ao formato do registro. |
| `declared_type` | `varchar(128)` | Não | MIME declarado; não usado como prova de validade. |
| `declared_size` | `integer` | Não | Bytes declarados: até 20M/500M/50M conforme tipo, proposta P-02. |
| `actual_size` | `bigint` | Sim | Bytes aferidos, inclusive tamanho inválido para diagnóstico da rejeição; limite do tipo é exigido antes de ready/valid. |
| `media_type` | `varchar(128)` | Sim | MIME efetivo normalizado. |
| `container` | `varchar(32)` | Sim | Container efetivo de catálogo validado. |
| `video_codec` | `varchar(64)` | Sim | Codec efetivo; quando houver trilha de vídeo. |
| `audio_codec` | `varchar(64)` | Sim | Codec efetivo; quando houver trilha de áudio. |
| `duration` | `numeric(12,6)` | Sim | Segundos; mídia temporal exige valor aferido, sem duração mínima de produto. |
| `frame_rate_numerator` | `integer` | Sim | Numerador da taxa de quadros original aferida; obrigatório para vídeo validado. |
| `frame_rate_denominator` | `integer` | Sim | Denominador positivo da taxa original; base para impedir aumento de FPS. |
| `width` | `integer` | Sim | Largura visual validada, quando aplicável. |
| `height` | `integer` | Sim | Altura visual validada, quando aplicável. |
| `orientation` | `smallint` | Sim | Orientação EXIF 1..8 para foto, quando presente. |
| `rotation` | `smallint` | Sim | Rotação de vídeo normalizada em 0/90/180/270, quando presente. |
| `sha256` | `bytea` | Sim | 32 bytes do conteúdo validado; ETag não substitui esse hash. |
| `validation_state` | `varchar(24)` | Não | expected, receiving, received, validating, valid ou rejected. |
| `processing_state` | `varchar(24)` | Não | not_requested, pending, processing, available ou failed. |
| `profile_version` | `integer` | Não | Default 1; contrato de variantes desejado. |
| `received_at` | `timestamp without time zone` | Sim | Recebimento integral por evidência do armazenamento. |
| `validated_at` | `timestamptz` | Sim | Conclusão da validação efetiva. |
| `confirmed_at` | `timestamp without time zone` | Sim | Confirmação junto do registro, não antes. |
| `error_code` | `varchar(80)` | Sim | Falha sanitizada de validação/processamento. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_files_records_id` | `files_records_id` → `records.records` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_files_kind` | kind válido; position em 1..10; profile_version >= 1. |
| `ck_files_size` | declared_size positivo e <= limite do tipo; actual_size, se presente, positivo. Arquivo valid exige actual_size = declared_size e limite do tipo; rejeitado pode registrar tamanho divergente ou excedente. |
| `ck_files_media` | Foto não tem duration; vídeo/áudio validados exigem duration >= 0; duração máxima por categoria é validada na regra agregada do registro. |
| `ck_files_validation` | validation_state e processing_state válidos; valid exige metadados aferidos, received_at, validated_at e hash de 32 bytes, container normalizado do catálogo inicial e codec para mídia temporal. |
| `ck_files_dimensions` | Dimensões positivas em foto/vídeo validado; orientation em 1..8 e rotation em {0,90,180,270}, quando presentes. |
| `ck_files_rate` | Taxa original em par racional positivo; vídeo validado exige taxa; foto/áudio não possuem taxa de vídeo. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_files_record_position` | `files_records_id`, `position` | Sim | Unicidade/invariante. |
| `uq_files_record_client` | `files_records_id`, `client_key` | Sim | Unicidade/invariante. |
| `ix_files_processing` | `processing_state`, `id` | Não | Reconciliação de preparação pendente. |

**Regras complementares**

- Não armazenar nome original do arquivo nem EXIF de localização em colunas de metadados.
- Um conjunto de fotos pode ter variantes com resultados diferentes após a confirmação integral; isso não desfaz o recebimento.
- Duração é validada antes de arredondamento/coerção; não truncar para enquadrar limite.

### `records.storage_objects`

Objeto S3 planejado ou persistido; fixa a versão física usada, sem guardar URL utilizável.

Requisitos: FR-011, FR-014, FR-020. PK: `pk_storage_objects_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `storage_objects_files_id` | `bigint` | Não | Arquivo proprietário. |
| `storage_objects_processing_attempts_id` | `bigint` | Sim | Tentativa que gerou objeto derivado; escopo do arquivo deve coincidir. |
| `kind` | `varchar(16)` | Não | temporary, original ou derived. |
| `state` | `varchar(24)` | Não | planned, initializing, uploading, stored, deleting ou deleted. |
| `bucket` | `varchar(63)` | Não | Bucket privado, separado por ambiente. |
| `object_key` | `text` | Não | Chave opaca gerada pelo servidor; 1..1024 bytes UTF-8. |
| `version_id` | `text` | Sim | Versão S3 opaca, fixada após recepção/persistência. |
| `multipart_id` | `text` | Sim | Identificador externo do multipart, quando houver; não é credencial. |
| `etag` | `text` | Sim | ETag opaco; não assumir que seja MD5 do conteúdo. |
| `size` | `bigint` | Sim | Bytes do objeto efetivo; positivo após stored. |
| `sha256` | `bytea` | Sim | 32 bytes quando calculado/verificado. |
| `last_modified_at` | `timestamptz` | Sim | Instante original informado pelo S3, usado como evidência. |
| `observed_at` | `timestamptz` | Sim | Instante em que o backend verificou a versão/recebimento. |
| `stored_at` | `timestamptz` | Sim | Persistência conhecida desta versão. |
| `deleted_at` | `timestamptz` | Sim | Remoção da versão confirmada. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_storage_objects_files_id` | `storage_objects_files_id` → `records.files` (`id`) |
| `fk_storage_objects_processing_attempts_file_id` | `storage_objects_files_id`, `storage_objects_processing_attempts_id` → `records.processing_attempts` (`processing_attempts_files_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_storage_objects_state` | kind/state válidos; stored exige version_id, size, last_modified_at, observed_at e stored_at; deleted exige deleted_at. |
| `ck_storage_objects_storage` | Bucket válido de 3..63 caracteres; object_key de 1..1024 bytes; size > 0; hash presente tem 32 bytes. |
| `ck_storage_objects_origin` | derived exige tentativa; temporary/original não apontam tentativa de conversão. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_storage_objects_object_key` | `bucket`, `object_key` | Sim | Unicidade/invariante. |
| `uq_storage_objects_file_scope` | `storage_objects_files_id`, `id` | Sim | Unicidade/invariante. |
| `uq_storage_objects_original` | `storage_objects_files_id` | Sim | Parcial: kind=original e state diferente de deleted. Unicidade/invariante. |
| `ix_storage_objects_attempt` | `storage_objects_processing_attempts_id` | Não | Parcial: storage_objects_processing_attempts_id preenchido. Localizar saídas de uma tentativa. |

**Regras complementares**

- Originais e derivados recebem novas chaves, sem sobrescrever objeto confirmado.
- Depois de fixada uma versão recebida, um PUT tardio na chave temporária não pode trocar a versão já selecionada.
- Evento de armazenamento atrasado não altera last_modified_at para o horário do consumidor.

### `records.file_variants`

Slot de variante de exibição/reprodução para um arquivo e perfil.

Requisitos: FR-018, FR-019, FR-020. PK: `pk_file_variants_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `file_variants_files_id` | `bigint` | Não | Arquivo do registro confirmado. |
| `file_variants_storage_objects_id` | `bigint` | Sim | Objeto derivado da mesma mídia; obrigatório quando ready. |
| `kind` | `varchar(16)` | Não | image, thumbnail ou playback. |
| `profile_version` | `integer` | Não | Versão do perfil que define essa saída. |
| `state` | `varchar(24)` | Não | pending, processing, ready ou failed. |
| `media_type` | `varchar(128)` | Sim | image/jpeg, video/mp4 ou audio/mp4 conforme saída. |
| `container` | `varchar(32)` | Sim | jpeg, mp4 ou m4a conforme contrato. |
| `video_codec` | `varchar(64)` | Sim | H.264 para playback de vídeo. |
| `audio_codec` | `varchar(64)` | Sim | AAC para vídeo/áudio quando houver trilha. |
| `width` | `integer` | Sim | Largura final de imagem/vídeo. |
| `height` | `integer` | Sim | Altura final de imagem/vídeo. |
| `duration` | `numeric(12,6)` | Sim | Duração final em segundos para playback temporal. |
| `frame_rate_numerator` | `integer` | Sim | Numerador racional da taxa de quadros, quando vídeo. |
| `frame_rate_denominator` | `integer` | Sim | Denominador positivo, quando vídeo. |
| `location_removed` | `boolean` | Sim | Obrigatoriamente true para versões de foto exibidas. |
| `ready_at` | `timestamptz` | Sim | Saída disponível. |
| `error_code` | `varchar(80)` | Sim | Motivo seguro de falha. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_file_variants_files_id` | `file_variants_files_id` → `records.files` (`id`) |
| `fk_file_variants_storage_objects_file_id` | `file_variants_files_id`, `file_variants_storage_objects_id` → `records.storage_objects` (`storage_objects_files_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_file_variants_state` | kind/state válidos; profile_version >= 1; ready exige objeto e ready_at. |
| `ck_file_variants_media` | Metadados presentes têm dimensões positivas/duração não negativa; location_removed, se presente, é booleano. Compatibilidade com o arquivo pai pertence à regra agregada ct_file_variants_ready. |
| `ck_file_variants_rate` | Numerador/denominador ambos nulos ou positivos; contrato de vídeo exige razão <= 30. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_file_variants_file_kind_profile` | `file_variants_files_id`, `kind`, `profile_version` | Sim | Unicidade/invariante. |
| `ix_file_variants_object` | `file_variants_storage_objects_id` | Não | Parcial: file_variants_storage_objects_id preenchido. Rastreio da variante física. |

**Regras complementares**

- Imagem até 2048 px no maior lado; miniatura até 400; não ampliar resolução menor.
- Vídeo até 1080p/30 fps, preservando proporção e sem ampliar fps; detalhes do perfil vertical dependem de validação.
- Não servir original como fallback de variante ausente ou como forma de contornar bloqueio.

### `records.processing_jobs`

Trabalho lógico idempotente de preparação de uma mídia.

Requisitos: FR-020. PK: `pk_processing_jobs_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `processing_jobs_files_id` | `bigint` | Não | Arquivo a preparar. |
| `kind` | `varchar(24)` | Não | photo_prepare ou av_transcode. |
| `profile_version` | `integer` | Não | Versão do perfil desejado. |
| `profile` | `jsonb` | Não | Snapshot técnico versionado do perfil, sem conteúdo, URL ou nomes pessoais. |
| `state` | `varchar(24)` | Não | pending, running, reconciling, completed ou failed. |
| `attempt_count` | `integer` | Não | Default 0; número de tentativas persistidas. |
| `next_attempt_at` | `timestamptz` | Não | Próxima tentativa/reconciliação elegível. |
| `completed_at` | `timestamptz` | Sim | Resultado final do trabalho. |
| `last_error_code` | `varchar(80)` | Sim | Falha sanitizada. |
| `lease_token` | `uuid` | Sim | Token da posse atual; escrita condicionada ao token. |
| `lease_until` | `timestamptz` | Sim | Prazo da posse; preenchido junto do token. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_processing_jobs_files_id` | `processing_jobs_files_id` → `records.files` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_processing_jobs_state` | kind/state válidos; profile_version >= 1; attempt_count >= 0; profile é objeto JSON; lease em par. |
| `ck_processing_jobs_completion` | completed/failed exigem completed_at; demais estados mantêm completed_at nulo. Conjunto de saídas é validado em ct_processing_jobs_completion. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_processing_jobs_file_profile` | `processing_jobs_files_id`, `kind`, `profile_version` | Sim | Unicidade/invariante. |
| `uq_processing_jobs_file_scope` | `processing_jobs_files_id`, `id` | Sim | Unicidade/invariante. |
| `ix_processing_jobs_eligible` | `next_attempt_at`, `id` | Não | Parcial: state em pending/reconciling/running. Reivindicar trabalho elegível. |

**Regras complementares**

- Concorrência limitada por tipo de mídia; job não termina até persistir todas as variantes previstas.
- Falha final mantém o original; retry/reprocessamento não cria novo registro.

### `records.processing_attempts`

Tentativa concreta local ou MediaConvert; preserva referências para reconciliar resultado externo desconhecido.

Requisitos: FR-020. PK: `pk_processing_attempts_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `processing_attempts_processing_jobs_id` | `bigint` | Não | Trabalho lógico. |
| `processing_attempts_files_id` | `bigint` | Não | Arquivo redundante para assegurar coerência de saídas. |
| `number` | `integer` | Não | Sequência de tentativa por job. |
| `request_key` | `uuid` | Não | Identidade estável antes de chamar o provedor; comportamento real de idempotência deve ser validado. |
| `provider` | `varchar(24)` | Não | local ou mediaconvert. |
| `provider_job_id` | `text` | Sim | Job ID opaco, quando conhecido. |
| `provider_job_hash` | `bytea` | Sim | SHA-256 de 32 bytes do identificador externo, índice fixo; comparar também o ID integral. |
| `output_prefix` | `text` | Não | Prefixo exclusivo de saída; sem PII, sem URL assinada. |
| `state` | `varchar(24)` | Não | created, submitted, running, succeeded, failed ou unknown. |
| `started_at` | `timestamptz` | Não | Início técnico. |
| `finished_at` | `timestamptz` | Sim | Fim observado. |
| `error_code` | `varchar(80)` | Sim | Código seguro. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_processing_attempts_processing_jobs_file_id` | `processing_attempts_files_id`, `processing_attempts_processing_jobs_id` → `records.processing_jobs` (`processing_jobs_files_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_processing_attempts_state` | provider/state válidos; number >= 1; succeeded/failed exigem finished_at; finished_at >= started_at. |
| `ck_processing_attempts_prefix` | output_prefix não vazio, sem URL; prefixo/chaves finais respeitam 1024 bytes. |
| `ck_processing_attempts_provider` | provider_job_id e provider_job_hash em par; hash possui 32 bytes; mediaconvert em submitted/running/succeeded exige ID externo conhecido. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_processing_attempts_job_number` | `processing_attempts_processing_jobs_id`, `number` | Sim | Unicidade/invariante. |
| `uq_processing_attempts_request` | `request_key` | Sim | Unicidade/invariante. |
| `uq_processing_attempts_provider_job` | `provider`, `provider_job_hash` | Sim | Parcial: provider_job_hash não nulo. Unicidade/invariante. |
| `uq_processing_attempts_file_scope` | `processing_attempts_files_id`, `id` | Sim | Unicidade/invariante. |
| `uq_processing_attempts_active` | `processing_attempts_processing_jobs_id` | Sim | Parcial: state em created/submitted/running/unknown. Unicidade/invariante. |

**Regras complementares**

- unknown impede nova submissão automática até reconciliação; idempotência local não promete exatamente um job externo.
- Callbacks atrasados só afetam a tentativa correspondente, sem regredir uma saída final válida.

### `records.cleanup_jobs`

Limpeza recuperável de temporários/rejeitados ou multipart incompleto.

Requisitos: FR-014, FR-020. PK: `pk_cleanup_jobs_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `cleanup_jobs_storage_objects_id` | `bigint` | Não | Objeto/transferência alvo. |
| `action` | `varchar(24)` | Não | delete_object ou abort_multipart. |
| `eligible_at` | `timestamptz` | Não | Marco + 24 horas, conforme classificação; sem apagar pendência que possa confirmar. |
| `state` | `varchar(24)` | Não | pending, running, reconciling, completed, failed ou blocked. |
| `attempt_count` | `integer` | Não | Default 0. |
| `next_attempt_at` | `timestamptz` | Não | Próxima tentativa. |
| `completed_at` | `timestamptz` | Sim | Conclusão da ação remota e persistência local. |
| `last_error_code` | `varchar(80)` | Sim | Código sanitizado, incluindo proteção de original confirmado. |
| `lease_token` | `uuid` | Sim | Token da posse atual; escrita condicionada ao token. |
| `lease_until` | `timestamptz` | Sim | Prazo da posse; preenchido junto do token. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_cleanup_jobs_storage_objects_id` | `cleanup_jobs_storage_objects_id` → `records.storage_objects` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_cleanup_jobs_state` | action/state válidos; attempt_count não negativo; lease em par; completed exige completed_at. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_cleanup_jobs_object_action` | `cleanup_jobs_storage_objects_id`, `action` | Sim | Unicidade/invariante. |
| `ix_cleanup_jobs_eligible` | `next_attempt_at`, `id` | Não | Parcial: state em pending/running/reconciling. Limpeza pendente com backoff. |

**Regras complementares**

- Validar novamente propriedade, versão S3 e ausência de proteção do objeto antes de cada efeito externo.
- Temporário de registro confirmado pode ser limpo após original durável; original/variante confirmados não são elegíveis à rotina de 24h.
- Objeto órfão de promoção com commit desconhecido exige reconciliação antes de exclusão.

## 5. Notificações — `notifications`

### `notifications.notifications`

Intenção durável de e-mail transacional, com deduplicação por obrigação de negócio.

Requisitos: FR-004, FR-017, FR-021. PK: `pk_notifications_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `notifications_events_id` | `bigint` | Não | Evento relacionado. |
| `notifications_accounts_id` | `bigint` | Sim | Destinatário conhecido; obrigatório para aviso de abertura. |
| `notifications_invitations_id` | `bigint` | Sim | Convite; FK composta também valida o evento. |
| `notifications_opening_dispatches_id` | `bigint` | Sim | Obrigação de abertura; FK composta também valida o evento. |
| `kind` | `varchar(24)` | Não | event_invitation ou event_opened. |
| `request_key` | `uuid` | Sim | Obrigatória para pedido interativo de e-mail de convite; gerada antes do POST e estável em retry. NULL para obrigação de abertura. |
| `deduplication_key` | `varchar(200)` | Não | Chave canônica: invitation:{convite}:{request_key} ou opening:{opening_dispatch}; sem e-mail em claro. |
| `template_version` | `integer` | Não | Versão positiva do template, sem corpo de e-mail no banco. |
| `access_version` | `bigint` | Sim | Versão da configuração de acesso; obrigatória no aviso de abertura. |
| `recipient_email_ciphertext` | `bytea` | Sim | E-mail de destino em envelope autenticado; não aparece no payload de Outbox. |
| `recipient_hash` | `bytea` | Sim | HMAC de 32 bytes para deduplicação do destino sem e-mail em claro. |
| `hash_key_id` | `varchar(64)` | Sim | Versão da chave de HMAC; necessária se recipient_hash existir. |
| `encryption_key_id` | `varchar(64)` | Sim | Chave do envelope; necessária se houver ciphertext. |
| `state` | `varchar(24)` | Não | pending, running, reconciling, accepted, failed ou cancelled. |
| `due_at` | `timestamptz` | Não | Elegibilidade técnica; abertura local convertida explicitamente para UTC. |
| `attempt_count` | `integer` | Não | Default 0; não negativo. |
| `next_attempt_at` | `timestamptz` | Não | Próxima avaliação, com backoff configurável. |
| `accepted_at` | `timestamptz` | Sim | Aceite pelo SES; não comprova entrega na caixa de entrada. |
| `last_error_code` | `varchar(80)` | Sim | Código de catálogo, sem resposta bruta do provedor. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Prazo técnico da posse; preenchido junto com lease_token. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_notifications_events_id` | `notifications_events_id` → `events.events` (`id`) |
| `fk_notifications_accounts_id` | `notifications_accounts_id` → `identity.accounts` (`id`) |
| `fk_notifications_invitations_id` | `notifications_events_id`, `notifications_invitations_id` → `events.invitations` (`invitations_events_id`, `id`) |
| `fk_notifications_opening_dispatches_id` | `notifications_events_id`, `notifications_opening_dispatches_id` → `events.opening_dispatches` (`opening_dispatches_events_id`, `id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_notifications_kind` | event_invitation exige convite e não possui obrigação de abertura; event_opened exige obrigação, conta e access_version positivo, sem convite. |
| `ck_notifications_state` | state do catálogo; attempt_count >= 0; template_version > 0; accepted exige accepted_at. |
| `ck_notifications_recipient` | Ciphertext e encryption_key_id em par; recipient_hash tem 32 bytes e acompanha hash_key_id; estado pronto para enviar exige destino protegido. |
| `ck_notifications_deduplication` | event_invitation exige request_key e chave canônica invitation:{convite}:{request_key}; event_opened exige request_key nulo e chave opening:{opening_dispatch}. A UNIQUE da chave impõe o escopo lógico sem duplicar índice. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_notifications_deduplication` | `deduplication_key` | Sim | Unicidade/invariante. |
| `ix_notifications_eligible` | `next_attempt_at`, `id` | Não | Parcial: state em pending/running/reconciling. Buscar intenções de envio elegíveis. |
| `ix_notifications_events` | `notifications_events_id`, `id` | Não | Rastrear obrigações por evento. |
| `ix_notifications_accounts` | `notifications_accounts_id` | Não | Parcial: notifications_accounts_id não nulo. Operação referencial e privacidade. |
| `ix_notifications_invitations` | `notifications_events_id`, `notifications_invitations_id` | Não | Parcial: notifications_invitations_id não nulo. Operação referencial do convite. |
| `ix_notifications_opening` | `notifications_events_id`, `notifications_opening_dispatches_id` | Não | Parcial: notifications_opening_dispatches_id não nulo. Operação referencial da obrigação. |

**Regras complementares**

- E-mail de convite é pedido explícito ao contrato de Notifications, após o convite existir; intenção, destino criptografado e Outbox são gravados localmente. Se a persistência falhar, repetir request_key sem recriar convite.
- No replay, comparar convite, template e destino normalizado usando o hash_key_id original; chave reutilizada com destinatário diferente é conflito. Abertura deduplica pelo opening_dispatch. Consultar Events/Identity por contratos antes de enviar.
- Persistir destinatário protegido antes do efeito externo; nunca guardar token em claro, corpo renderizado ou URL utilizável. Aviso obsoleto é cancelado.
- accepted gera Outbox de conclusão; Events atualiza opening_dispatches em sua própria transação. SES não participa da transação SQL.

### `notifications.notification_attempts`

Tentativas e respostas SES; separa aceite, entrega e resultado desconhecido.

Requisitos: FR-021, FR-022. PK: `pk_notification_attempts_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `notification_attempts_notifications_id` | `bigint` | Não | Intenção de notificação. |
| `number` | `integer` | Não | Número positivo, sequencial por intenção sob lock local. |
| `request_key` | `uuid` | Não | Identificação local estável da tentativa; não pressupõe deduplicação SES. |
| `provider_message_id` | `text` | Sim | Identificador opaco devolvido pelo SES; tamanho sujeito ao contrato do adaptador. |
| `provider_message_hash` | `bytea` | Sim | SHA-256 de 32 bytes do ID externo, para índice de tamanho fixo; comparar também ID integral. |
| `recipient_email_ciphertext` | `bytea` | Sim | Destino efetivamente usado; envelope protegido, limpeza conforme retenção. |
| `encryption_key_id` | `varchar(64)` | Sim | Chave do envelope, se houver destinatário. |
| `state` | `varchar(24)` | Não | prepared, submitting, accepted, failed ou unknown. |
| `delivery_state` | `varchar(24)` | Sim | pending, delivered, bounced ou complained; feedback não desfaz aceite. |
| `started_at` | `timestamptz` | Não | Preparação persistida antes de chamar SES. |
| `finished_at` | `timestamptz` | Sim | Conclusão local, quando conhecida. |
| `feedback_at` | `timestamptz` | Sim | Momento do último feedback autenticado. |
| `error_code` | `varchar(80)` | Sim | Código normalizado, sem conteúdo sensível. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_notification_attempts_notifications_id` | `notification_attempts_notifications_id` → `notifications.notifications` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_notification_attempts_state` | Estado e delivery_state do catálogo; number > 0; accepted exige provider_message_id, hash e finished_at; finished_at >= started_at quando preenchido. |
| `ck_notification_attempts_provider` | ID externo e hash em par; hash possui 32 bytes; destinatário criptografado e chave em par. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_notification_attempts_number` | `notification_attempts_notifications_id`, `number` | Sim | Unicidade/invariante. |
| `uq_notification_attempts_request` | `request_key` | Sim | Unicidade/invariante. |
| `uq_notification_attempts_provider` | `provider_message_hash` | Sim | Parcial: provider_message_hash não nulo. Unicidade/invariante. |
| `uq_notification_attempts_active` | `notification_attempts_notifications_id` | Sim | Parcial: state em prepared/submitting/unknown. Unicidade/invariante. |

**Regras complementares**

- Somente uma tentativa ativa por intenção. Timeout após chamada pode significar aceite: persistir unknown e reconciliar; repetição externa pode duplicar e-mail.
- Notificações SNS/SES são autenticadas e deduplicadas na inbox; feedback atrasado não regride delivered para pending. Bounce/complaint têm precedência operacional e não conferem acesso.
- Dados técnicos e destino protegido têm retenção proposta de até 180 dias; conservar a chave mínima da obrigação enquanto necessário para evitar novo envio lógico.

## 6. Estruturas técnicas por módulo

Os três contratos abaixo são instanciados **separadamente em identity, events, records e notifications**. A tabela exemplificada usa o schema identity; substituir somente o schema nas outras instâncias. Colunas, constraints e índices conservam os mesmos nomes locais, válidos por schema. Não há tabela técnica central nem commit conjunto entre contextos.

### `identity.outbox_messages`

Outbox local de identity: intenção atômica com a alteração de domínio.

Requisitos: Transversal; FR-014, FR-016, FR-020, FR-021, FR-022. PK: `pk_outbox_messages_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `message_id` | `uuid` | Não | Identificador estável do contrato; deduplicação não usa ID SQS. |
| `type` | `varchar(100)` | Não | Nome de evento/comando, catálogo versionado. |
| `protocol_version` | `smallint` | Não | Versão positiva do contrato. |
| `aggregate_type` | `varchar(64)` | Sim | Tipo lógico do agregado, sem referência polimórfica física. |
| `aggregate_id` | `bigint` | Sim | ID lógico; não é FK por abranger tipos e retenções distintas. |
| `payload` | `jsonb` | Não | Objeto com IDs e metadados permitidos; sem nomes, e-mails, textos, credenciais ou URLs assinadas. |
| `correlation_id` | `uuid` | Não | Correlação ponta a ponta. |
| `causation_id` | `uuid` | Sim | Mensagem causadora, quando houver; referência lógica. |
| `trace_context` | `jsonb` | Não | Default {}; allowlist de contexto W3C, sem baggage sensível. |
| `occurred_at` | `timestamptz` | Não | Instante original do fato. |
| `state` | `varchar(24)` | Não | pending, publishing, published ou failed. |
| `attempt_count` | `integer` | Não | Default 0; não negativo. |
| `available_at` | `timestamptz` | Não | Elegibilidade da próxima publicação. |
| `published_at` | `timestamptz` | Sim | Confirmação local da publicação; timeout pode causar publicação repetida. |
| `last_error_code` | `varchar(80)` | Sim | Código sem resposta/payload bruto. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Prazo técnico da posse; preenchido junto com lease_token. |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_outbox_messages_state` | state válido; attempt_count >= 0; protocol_version > 0; published exige published_at; aggregate_type e aggregate_id em par. |
| `ck_outbox_messages_payload` | payload e trace_context são objetos JSON, com allowlist e limites de contrato; sem dados sensíveis. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_outbox_messages_message` | `message_id` | Sim | Unicidade/invariante. |
| `ix_outbox_messages_eligible` | `available_at`, `id` | Não | Parcial: state em pending/publishing. Claim em lote; comparar relógio fora do predicado. |
| `ix_outbox_messages_retention` | `published_at`, `id` | Não | Parcial: state = published. Expurgo técnico por lotes. |

**Regras complementares**

- Gravar na mesma transação local do domínio. Claim curto com lease; publicar fora da transação; concluir somente com token de posse atual.
- Publicação é ao menos uma vez; mensagem repetida mantém message_id. Não apagar pendências por idade nem converter failed em sucesso.
- Retenção técnica de 180 dias para concluídas; pendências e DLQ exigem resolução. Payload deve ser objeto e ter limite de bytes de contrato proposto de 16 KiB (P-04).

### `identity.processed_messages`

Inbox/deduplicação local de consumidores de identity.

Requisitos: Transversal; FR-014, FR-016, FR-020, FR-021, FR-022. PK: `pk_processed_messages_id`. Metadados: padrão mutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `consumer` | `varchar(80)` | Não | Nome estável do consumidor lógico, independente da réplica. |
| `source` | `varchar(32)` | Não | Catálogo do adaptador: internal, s3, mediaconvert, ses ou scheduler. |
| `message_id` | `uuid` | Sim | Identificador interno; exclusivo com external_id. |
| `external_id` | `text` | Sim | Identidade estável autenticada do evento externo; não usar receipt handle. |
| `external_hash` | `bytea` | Sim | SHA-256 de 32 bytes do external_id para unicidade; comparar texto integral no replay. |
| `payload_hash` | `bytea` | Não | SHA-256 de 32 bytes do contrato canônico; detecta mesmo ID com outro conteúdo. |
| `correlation_id` | `uuid` | Sim | Correlação, quando presente em fonte confiável. |
| `state` | `varchar(24)` | Não | processing, completed ou quarantined. |
| `attempt_count` | `integer` | Não | Default 0; não negativo. |
| `completed_at` | `timestamptz` | Sim | Commit do efeito/intenção local durável. |
| `error_code` | `varchar(80)` | Sim | Motivo normalizado de quarentena/falha. |
| `lease_token` | `uuid` | Sim | Token de posse/fencing; sem função de credencial. |
| `lease_until` | `timestamptz` | Sim | Prazo técnico da posse; preenchido junto com lease_token. |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_processed_messages_identity` | Exatamente um entre message_id e external_id; external_id acompanha external_hash de 32 bytes; payload_hash tem 32 bytes; consumer/source não vazios. |
| `ck_processed_messages_state` | state válido; attempt_count >= 0; completed exige completed_at; quarentena exige error_code. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `uq_processed_messages_internal` | `consumer`, `source`, `message_id` | Sim | Parcial: message_id não nulo. Unicidade/invariante. |
| `uq_processed_messages_external` | `consumer`, `source`, `external_hash` | Sim | Parcial: external_hash não nulo. Unicidade/invariante. |
| `ix_processed_messages_lease` | `lease_until`, `id` | Não | Parcial: state = processing. Recuperar trabalho interrompido. |
| `ix_processed_messages_retention` | `completed_at`, `id` | Não | Parcial: state = completed. Expurgo por lotes de consumos concluídos. |

**Regras complementares**

- Inbox e efeito local, ou criação do job durável local, compartilham transação. ACK SQS somente após commit; não aguardar conclusão externa dentro do lock.
- Unicidade é por consumidor/fonte/identidade. Mesmo ID e payload incompatível vai para quarentena; não ignorar silenciosamente.
- Eventos externos precisam de identidade e ordenação extraídas pelo adaptador; hash de conteúdo genérico não comprova ordem. Business keys mantêm a proteção após expurgo técnico em 180 dias.
- Reivindicar lease vencida com novo token; trabalhador antigo não pode concluir. Replays não regridem confirmação, travamento ou estado final.

### `identity.audit_entries`

Auditoria imutável de alterações sensíveis de identity.

Requisitos: FR-002, FR-004, FR-016, FR-017, FR-022. PK: `pk_audit_entries_id`. Metadados: padrão imutável.

| Coluna | Tipo PostgreSQL | Nulo? | Semântica / default / limite |
| --- | --- | --- | --- |
| `audit_entries_accounts_id` | `bigint` | Sim | Autor autenticado, quando houver; FK ao tombstone de conta. |
| `audit_entries_events_id` | `bigint` | Sim | Evento afetado, quando aplicável. |
| `actor_type` | `varchar(16)` | Não | account, guest ou system; guest não recebe identidade permanente. |
| `action` | `varchar(80)` | Não | Ação de catálogo. |
| `subject_type` | `varchar(64)` | Não | Tipo de objeto alterado. |
| `subject_id` | `bigint` | Sim | Referência lógica; sem FK polimórfica. |
| `outcome` | `varchar(16)` | Não | success, rejected ou failure. |
| `reason_code` | `varchar(80)` | Sim | Motivo normalizado. |
| `correlation_id` | `uuid` | Não | Correlação para investigação. |
| `before_state` | `jsonb` | Não | Default {}; allowlist de estados/IDs/datas, sem conteúdo ou identidade do convidado. |
| `after_state` | `jsonb` | Não | Default {}; mesma política de minimização. |
| `expires_at` | `timestamptz` | Não | created_at + 180 dias; retenção do PRD. |

**Relacionamentos físicos**

| Constraint | Origem → destino |
| --- | --- |
| `fk_audit_entries_accounts_id` | `audit_entries_accounts_id` → `identity.accounts` (`id`) |
| `fk_audit_entries_events_id` | `audit_entries_events_id` → `events.events` (`id`) |

**CHECKs da linha**

| Constraint | Regra declarativa |
| --- | --- |
| `ck_audit_entries_actor` | actor_type e outcome válidos; account exige FK de conta; guest/system não possuem autor de conta; expires_at = created_at + 180 dias. |
| `ck_audit_entries_payload` | before_state/after_state são objetos de allowlist; action e subject_type não vazios; sem conteúdo sensível. |

**Chaves candidatas e índices**

| Nome | Colunas na ordem | Único? | Filtro / finalidade |
| --- | --- | --- | --- |
| `ix_audit_entries_accounts` | `audit_entries_accounts_id`, `created_at`, `id` | Não | Parcial: audit_entries_accounts_id não nulo. Investigar ação autenticada/operar referência. |
| `ix_audit_entries_events` | `audit_entries_events_id`, `created_at`, `id` | Não | Parcial: audit_entries_events_id não nulo. Histórico de mudanças de acesso/evento. |
| `ix_audit_entries_retention` | `expires_at`, `id` | Não | Expurgo de auditoria em lotes. |

**Regras complementares**

- Somente append e expurgo autorizado por retenção. Mudança sensível e auditoria de sucesso são gravadas atomicamente no módulo dono.
- Tentativas negadas podem ser registradas em transação própria após rollback, sem converter falha em sucesso.
- Metadados de mídia, texto, nomes de convidados, e-mails, tokens e URLs não entram em before/after. Logs/alertas seguem a mesma proteção.

## 7. Integridade agregada e transições

As regras abaixo orientam os constraint triggers diferíveis no módulo dono, complementados por guardas imediatas, locks, escrita condicional e privilégios restritos. A base Liquibase implementa a validação do estado final da transação nas tabelas participantes, inclusive UPDATE/DELETE no filho, sem consultar serviços externos. A aplicação continua responsável por autorização, coordenação dos efeitos externos e comparação da versão esperada.

| Nome proposto | Tabelas do mesmo módulo | Invariante |
| --- | --- | --- |
| `ct_events_members` | `events.events`, `events.event_members` | Ao fim da criação existe slot 1 ligado à conta criadora; todo evento tem 1..2 membros. Slot 1 e ausência de remoção mantêm o mínimo; verificar novamente no commit de qualquer manutenção excepcional. |
| `ct_events_cover` | `events.events`, `events.covers` | Capa selecionada pertence ao evento, está ready e sua versão é fixada. Impedir alteração estrutural após locked_at, inclusive troca indireta do objeto de capa; ready não pode ser retirado enquanto selecionado. |
| `ct_upload_sessions_operation` | `records.records`, `records.idempotent_operations`, `records.upload_sessions` | Registro, operação e sessão são 1:1, do mesmo evento; a operação ligada pela sessão aponta exatamente para seu record, e started_at = submitted_at. Criação e conclusão desse conjunto são atômicas. |
| `ct_records_confirmation` | `records.records`, `records.files`, `records.storage_objects`, `records.upload_sessions`, `records.idempotent_operations`, `records.processing_jobs` | Confirmar somente com texto válido sem arquivos, ou exatamente expected_file_count arquivos válidos do mesmo formato, posições contínuas 1..n, originais stored imutáveis e metadados correspondentes. Recebimento de todos <= deadline do snapshot; recebimento do grupo é o máximo dos arquivos, texto usa o recebimento do request. Duração máxima: vídeo testimonial 300s/moment 60s, áudio testimonial 300s. Sessão/operação/registro, jobs de mídia e Outbox são persistidos juntos; confirmação e identidade de conteúdo nunca regridem. |
| `ct_storage_objects_confirmed` | `records.records`, `records.files`, `records.storage_objects`, `records.file_variants` | Não alterar/excluir versão de original de registro confirmado nem permitir transição deleting/deleted por limpeza comum; derived pronto selecionado também é protegido. Distinguir temporário removível e objeto de promoção com resultado desconhecido. A regra garante metadados no banco, não existência física no S3. |
| `ct_file_variants_ready` | `records.files`, `records.storage_objects`, `records.file_variants`, `records.processing_attempts`, `records.processing_jobs` | ready exige objeto derived/stored do mesmo arquivo e tentativa/perfil elegíveis, com metadados reais. Foto: JPEG, imagem <=2048px no maior lado, thumbnail <=400px, sem ampliar menor, sem localização e com orientação/proporção preservadas. Vídeo: MP4/H.264/AAC, perfil 1080p/30fps com dimensões/rotação definidas, sem upscale. Áudio: M4A/AAC. Relação com mídia pai não é CHECK. |
| `ct_processing_jobs_completion` | `records.files`, `records.file_variants`, `records.processing_jobs`, `records.processing_attempts` | Job só completed quando todas as variantes exigidas pelo snapshot profile e profile_version estão ready. Sucesso atrasado de outra tentativa/perfil não substitui saída selecionada nem regride o estado; erro de conversão preserva confirmed_at e original. |

Limites de tipo/tamanho e catálogo da própria linha ficam em CHECKs. A decodificação de mídia, presença física do objeto, confirmação Cognito, relógio e autorização são verificações de aplicação/adaptadores. O trigger valida o estado persistido dessas verificações; não chama AWS.

Todas as transições descritas na [modelagem](DB-Modelagem.md#estados-e-transições) usam estado esperado/versão ou lock. Não tratar o catálogo de estados de um CHECK como garantia de transição válida.

## 8. Escopo, consultas e manutenção

- FKs compostas materializam a pertença de capa, emissor, destinatário, registro, operação, objeto e tentativa ao mesmo escopo. Elas não concedem permissão para acessar outro evento nem para consultar outro DbContext.
- `events.events.first_record_id` é referência **lógica** a `records.records.id`; preserva o fato do travamento mesmo em procedimento extraordinário de privacidade. `aggregate_id`, `subject_id` e `causation_id` também são referências lógicas, sem FK polimórfica.
- Índices de galeria são parciais sobre confirmação, com `submitted_at, id` estáveis. A posição do arquivo vem do UNIQUE `files_records_id, position`. Não persistir uma segunda tabela de galeria ou contagens de convidados.
- Índices de fila filtram apenas estados estáveis e comparam prazo na consulta. Claim por lote usa transação curta; rede acontece depois do commit. SKIP LOCKED é restrito a filas, sem uso para autorização ou leitura de galeria. [SELECT PostgreSQL](https://www.postgresql.org/docs/18/sql-select.html).
- Não adicionar GIN, índices por nome de convidado, partições, busca textual, índices de cada coluna ou materializações preventivas. Reavaliar os índices propostos com planos e volume representativo antes de fixar migrations; esta documentação não prova desempenho.
- Para purge por FK composta, os índices já propostos por escopo ou ponteiro permitem localizar filhos; avaliar seletividade e ordem real no plano. A remoção é excepcional/por retenção e respeita RESTRICT e a ordem dos filhos.
- A ordem futura de criação separa schemas/tabelas de FKs e triggers. O ciclo `events ↔ covers` é resolvido com ponteiro de capa atual nulo na criação e constraints após ambas as tabelas. Jobs/tentativas precedem referências de objetos derivados; concluir uma variante ocorre depois de armazenar sua saída. Não introduzir transação distribuída para resolver dependências de definição.

A implementação SQL está na [base Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md), seguindo existência **e compatibilidade**, concorrência e rollback do [guideline SQL](../guidelines/sql.md). A reversão inicial é permitida somente com tabelas vazias e escritores interrompidos; não há remoção de dados ou CASCADE.
