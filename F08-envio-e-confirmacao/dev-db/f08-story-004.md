# F08-STORY-004 — Manifesto multipart e evidência versionada

O changeset `f08-story-004-video-multipart` acrescenta `files.multipart_checksums` como vetor ordenado de bytea. As partes começam na posição 1. O manifesto multipart tem entre duas e trinta entradas presentes e pertence a um arquivo de vídeo. A aplicação confere o tamanho, a quantidade exata e os 32 bytes de cada digest. O manifesto de PUT mantém a coluna nula. O identificador multipart continua em `storage_objects.multipart_id`.

A tabela `photo_upload_receipts` passa a aceitar `complete_multipart` junto de `put`. Preserva histórico, versões e permissões. A reversão preserva comprovantes PUT e é recusada se houver manifesto multipart, identificador de multipart, comprovante de conclusão ou divergência no catálogo. [Modelo da F08](../spec.md#6-modelo-de-dados)

A suíte do commit `e2c9364` passou com 29 testes em PostgreSQL 18.6 e Liquibase 4.33.0. O primeiro ensaio teve uma falha intermitente em uma regressão de rollback do catálogo anterior à story. A regressão passou isoladamente e na repetição completa.
