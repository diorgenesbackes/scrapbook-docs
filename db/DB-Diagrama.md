# Diagramas de entidade e relacionamento

[Modelagem](DB-Modelagem.md) · [Dicionário físico](DB-Modelo-Fisico.md) · [Índice](README.md)

Data inicial: **2026-11-11**. Diagramas do modelo de referência. Scripts correspondentes foram criados em **2026-11-12** no [componente Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md). O dicionário é a referência completa de colunas, FKs compostas, CHECKs e índices; os diagramas exibem somente um recorte dos atributos.

## Visão de negócio

```mermaid
erDiagram
    accounts ||--o{ event_members : participa
    events ||--|{ event_members : possui_1_ou_2
    events ||--o{ invitations : oferece
    events ||--o{ covers : personaliza
    events ||--o{ opening_dispatches : agenda_aviso
    events ||--o{ records : recebe
    consent_notices ||--o{ records : concordancia
    records ||--|| idempotent_operations : identifica_tentativa
    records ||--|| upload_sessions : autoriza_recebimento
    records ||--o{ files : zero_texto_ou_1_a_10
    files ||--o{ storage_objects : armazena
    files ||--o{ file_variants : disponibiliza
    files ||--o{ processing_jobs : prepara
    processing_jobs ||--o{ processing_attempts : tenta
    storage_objects ||--o{ cleanup_jobs : limpa_se_elegivel
    events ||--o{ notifications : comunica
    notifications ||--o{ notification_attempts : encaminha
```

As relações 1:1 de registro/operação/sessão são invariantes da transação local, além de UNIQUE/FK. Limites 1..2 e 1..10 são mais restritos que a notação “muitos”. Um arquivo confirmado possui um original durável; os vários objetos incluem temporários e derivados. Notificação de abertura é por responsável/versão; não por registro.

## Relações físicas por módulo

Prefixos de entidade identificam o schema, por exemplo `records_files` = `records.files`. Atributos exibidos são um recorte. Linhas abaixo representam FKs físicas e sua opcionalidade; relações lógicas e invariantes agregadas não são convertidas em FKs artificiais.

### Contas e Acesso — `identity`

```mermaid
erDiagram
    identity_accounts {
        bigint id PK
        string subject
        string name
        string state
    }
    identity_sessions {
        bigint id PK
        string state
    }
    identity_oidc_requests {
        bigint id PK
        string state
    }
    identity_account_operations {
        bigint id PK
        string kind
        string state
    }
    identity_accounts ||--o{ identity_sessions : sessions_accounts_id
    identity_accounts ||--o{ identity_account_operations : account_operations_accounts_id
```

### Eventos e Convites — `events`

```mermaid
erDiagram
    events_events {
        bigint id PK
        string code
        string name
        string opening_mode
        string access_state
    }
    events_event_members {
        bigint id PK
        smallint slot
    }
    events_invitations {
        bigint id PK
        string state
    }
    events_covers {
        bigint id PK
        string state
    }
    events_opening_dispatches {
        bigint id PK
        string state
    }
    identity_accounts {
        bigint id PK
        string subject
        string name
        string state
    }
    identity_accounts ||--o{ events_events : events_creator_accounts_id
    events_events ||--o{ events_event_members : event_members_events_id
    identity_accounts ||--o{ events_event_members : event_members_accounts_id
    events_events ||--o{ events_invitations : invitations_events_id
    events_event_members ||--o{ events_invitations : invitations_event_members_event_id
    identity_accounts o|--o{ events_invitations : invitations_recipient_accounts_id
    events_events ||--o{ events_covers : covers_events_id
    events_covers o|--o| events_events : events_covers_event_id
    events_events ||--o{ events_opening_dispatches : opening_dispatches_events_id
    events_event_members ||--o{ events_opening_dispatches : opening_dispatches_event_members_event_id
```

### Registros e Mídias — `records`

```mermaid
erDiagram
    records_consent_notices {
        bigint id PK
        string code
    }
    records_records {
        bigint id PK
        string category
        string format
        timestamp confirmed_at
        string state
    }
    records_idempotent_operations {
        bigint id PK
        uuid key
        string state
    }
    records_upload_sessions {
        bigint id PK
        string state
    }
    records_files {
        bigint id PK
        smallint position
        string kind
        string validation_state
        string processing_state
    }
    records_storage_objects {
        bigint id PK
        string kind
        string state
    }
    records_file_variants {
        bigint id PK
        string kind
        string state
    }
    records_processing_jobs {
        bigint id PK
        string kind
        string state
    }
    records_processing_attempts {
        bigint id PK
        string state
    }
    records_cleanup_jobs {
        bigint id PK
        string state
    }
    events_events {
        bigint id PK
        string code
        string name
        string opening_mode
        string access_state
    }
    events_events ||--o{ records_records : records_events_id
    records_consent_notices ||--o{ records_records : records_consent_notices_id
    events_events ||--o{ records_idempotent_operations : idempotent_operations_events_id
    records_records ||--o| records_idempotent_operations : idempotent_operations_records_event_id
    events_events ||--o{ records_upload_sessions : upload_sessions_events_id
    records_records ||--o| records_upload_sessions : upload_sessions_records_event_id
    records_idempotent_operations ||--o| records_upload_sessions : upload_sessions_idempotent_operations_event_id
    records_records ||--o{ records_files : files_records_id
    records_files ||--o{ records_storage_objects : storage_objects_files_id
    records_files ||--o{ records_file_variants : file_variants_files_id
    records_storage_objects o|--o{ records_file_variants : file_variants_storage_objects_file_id
    records_files ||--o{ records_processing_jobs : processing_jobs_files_id
    records_processing_jobs ||--o{ records_processing_attempts : processing_attempts_processing_jobs_file_id
    records_processing_attempts o|--o{ records_storage_objects : storage_objects_processing_attempts_file_id
    records_storage_objects ||--o{ records_cleanup_jobs : cleanup_jobs_storage_objects_id
```

### Notificações — `notifications`

```mermaid
erDiagram
    notifications_notifications {
        bigint id PK
        string kind
        string state
    }
    notifications_notification_attempts {
        bigint id PK
        string state
    }
    events_events {
        bigint id PK
        string code
        string name
        string opening_mode
        string access_state
    }
    identity_accounts {
        bigint id PK
        string subject
        string name
        string state
    }
    events_invitations {
        bigint id PK
        string state
    }
    events_opening_dispatches {
        bigint id PK
        string state
    }
    events_events ||--o{ notifications_notifications : notifications_events_id
    identity_accounts o|--o{ notifications_notifications : notifications_accounts_id
    events_invitations o|--o{ notifications_notifications : notifications_invitations_id
    events_opening_dispatches o|--o{ notifications_notifications : notifications_opening_dispatches_id
    notifications_notifications ||--o{ notifications_notification_attempts : notification_attempts_notifications_id
```

## Estruturas técnicas locais

O padrão abaixo existe em cada um dos quatro schemas, totalizando doze tabelas. Os IDs de agregado/assunto nas estruturas técnicas são referências lógicas tipadas; não há FK polimórfica ou transação global.

```mermaid
flowchart LR
    L[Transacao do modulo] --> D[Dados de dominio]
    L --> A[audit_entries]
    L --> O[outbox_messages]
    O --> P[Publicador com lease]
    P --> Q[SNS e SQS]
    Q --> I[processed_messages no modulo destinatario]
    I --> T[Transacao local do destinatario]
    T --> E[Efeito local ou job duravel]
    T --> N[Outbox local quando necessaria]
```

A publicação pode repetir. Inbox e efeito local compartilham commit, com ACK posterior. Chamada S3/Cognito/MediaConvert/SES fica fora da transação; persistir intenção, tentativa e estado incerto para reconciliação.

## Travamento eventual e autorização

```mermaid
sequenceDiagram
    participant R as Records
    participant B as PostgreSQL Records
    participant Q as Outbox / SNS / SQS
    participant E as Events
    participant V as API de visualizacao
    R->>B: Confirmar registro + fato + Outbox (commit local)
    B-->>R: Confirmado de forma duravel
    B->>Q: Publicar com message_id estavel
    Q->>E: Consumir fato, possivelmente repetido
    E->>E: Inbox + travamento + auditoria (commit local)
    V->>E: Consultar vinculo e politica atual por contrato
    E-->>V: Permissao conforme estado e horario de Brasilia
```

Entre confirmação e consumo existe a janela eventual aceita no HLD (meta de 5 segundos em condições normais). `events.events.first_record_id` é referência lógica, sem FK; uma manutenção excepcional no registro não reabre o evento. O Scheduler de e-mail não participa da autorização.
