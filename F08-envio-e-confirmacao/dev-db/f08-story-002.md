# F08-STORY-002 — Evidência de recepção por versão

O changeset `f08-story-002-photo-receipts` acrescenta o instante e o tipo de evidência a um objeto temporário observado e guarda os `VersionId` recebidos em `photo_upload_receipts`. A tabela registra o instante de criação informado pelo armazenamento e o marco de elegibilidade da limpeza. A role da aplicação pode gravar e consultar evidências e atualizar o horário de exclusão. O rollback bloqueia a remoção na presença de evidência ou divergência de catálogo. [Modelo](../spec.md#6-modelo-de-dados)

A suíte descartável do Liquibase 4.33.0 com PostgreSQL 18.6 exercitou 26 cenários, inclusive apply, rollback, grants e proteção contra divergência. O changeset foi preparado no commit `c8813f9`. A validação do estado confirmado no banco pertence ao ensaio HTTP e de armazenamento. [Plano de aceite](../test-funcional/2026-11-25_14-54-00/1-plano.md)
