# F10-STORY-001 — Evolução do schema

O changeset `f10-story-001-media-preparation` parte da geração `f08-story-004-video-multipart`. Acrescenta ao trabalho a identidade, correlação e instante da confirmação, junto ao contador de ciclos. A tentativa recebe a coluna para fixar o primeiro instante de submissão ao provedor. Esses campos deixam o contexto disponível durante todo o ciclo de processamento, inclusive se a Outbox tiver sido arquivada. [Modelo físico](../../db/DB-Modelo-Fisico.md#recordsprocessing_jobs) · [Spec da F10](../spec.md#6-modelo-de-dados)

O changeset aceita os trabalhos anteriores com contexto nulo e ciclo 1. A role de aplicação continua com DML. O rollback sob lock verifica a assinatura de catálogo e recusa remoção de evidências novas. Com execução ativa, preserve a evolução de schema ao reverter o binário.

A suíte Liquibase 4.33.0 com PostgreSQL 18.6 descartável concluiu 31 testes. Os cenários da F10 cobriram instalação, reaplicação, upgrade com registros F08, reversão compatível, preservação de evidências e recusa de DDL à role de aplicação.
