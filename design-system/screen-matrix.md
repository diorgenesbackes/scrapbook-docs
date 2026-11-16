# Matriz de telas e requisitos

[Índice](README.md) · [Inventário executável](screens.json)

38 superfícies (35 telas/mensagens + 3 diálogos), 178 estados explícitos. As rotas do catálogo são IDs de revisão; não constituem endpoints/rotas de produção. Layout, hierarquia, estados, responsivo e destinos estão na especificação individual.

| ID / especificação | Requisitos | Público / shell | Estados |
| --- | --- | --- | --- |
| [S01 — Boas-vindas](screens/S01.md) | FR-001, FR-003 | shared / auth | default, loading, error |
| [S02 — Entrar](screens/S02.md) | FR-001 | account / auth | default, loading, error, expired, rate-limit |
| [S03 — Criar conta](screens/S03.md) | FR-001 | account / auth | default, loading, error, success |
| [S04 — Confirmar e-mail](screens/S04.md) | FR-001, FR-002 + HLD | account / auth | default, loading, error, expired, success |
| [S05 — Recuperar acesso](screens/S05.md) | FR-002 + HLD | account / auth | default, loading, error, rate-limit, success |
| [S06 — Definir nova senha](screens/S06.md) | FR-002 + HLD | account / auth | default, loading, error, expired, success |
| [S07 — Minha conta](screens/S07.md) | FR-002 | account / account | default, loading, error, success, expired |
| [S08 — Retorno da autenticação](screens/S08.md) | FR-001 + HLD | account / auth | loading, error, expired, success |
| [S09 — Meus eventos](screens/S09.md) | FR-003 | owner / owner | default, loading, empty, error, expired |
| [S10 — Criar evento](screens/S10.md) | FR-003, FR-017 | owner / form | default, loading, error, success, uncertain |
| [S11 — Configuração do evento](screens/S11.md) | FR-003, FR-016, FR-017 | owner / owner | default, loading, error, locked, conflict, uncertain, success |
| [S12 — Personalizar capa](screens/S12.md) | FR-003, FR-011, FR-016 | owner / form | default, loading, error, locked, success |
| [S13 — Convidar responsável](screens/S13.md) | FR-004, FR-016 | owner / owner | default, loading, error, locked, success |
| [S14 — Aceitar convite](screens/S14.md) | FR-004, FR-016 | account / auth | default, loading, error, expired, used, replaced, full, locked, success |
| [S15 — Compartilhar evento e imprimir QR](screens/S15.md) | FR-005 | owner / owner | default, loading, error, print |
| [S16 — Página dos convidados](screens/S16.md) | FR-005, FR-006 | guest / guest | default, loading, error, before, closed |
| [S17 — Identificação e concordância](screens/S17.md) | FR-007 | guest / guest | default, error, returning, changed-name |
| [S18 — Escolher depoimento](screens/S18.md) | FR-008, FR-011 | guest / guest | default, loading, error, closed |
| [S19 — Escolher momento](screens/S19.md) | FR-009, FR-011 | guest / guest | default, loading, error, closed |
| [S20 — Fotografar ou selecionar fotos](screens/S20.md) | FR-008, FR-009, FR-010, FR-011 | guest / capture | default, permission, unsupported, error, selected, limit |
| [S21 — Gravar ou selecionar vídeo](screens/S21.md) | FR-008, FR-009, FR-010, FR-011 | guest / capture | default, recording, permission, unsupported, error, limit |
| [S22 — Gravar ou selecionar áudio](screens/S22.md) | FR-008, FR-010, FR-011 | guest / capture | default, recording, permission, unsupported, error, limit |
| [S23 — Escrever depoimento](screens/S23.md) | FR-008, FR-011, FR-012 | guest / guest | default, near-limit, limit, error |
| [S24 — Adicionar legenda](screens/S24.md) | FR-011, FR-012 | guest / guest | default, expanded, near-limit, limit, error |
| [S25 — Revisar antes de enviar](screens/S25.md) | FR-007, FR-013, FR-014 | guest / guest | default, preview-error, error, closed |
| [S26 — Enviar e confirmar recebimento](screens/S26.md) | FR-006, FR-009, FR-014 | guest / guest | default, progress, validating, offline, error, uncertain, deadline, success |
| [S27 — Recebimento confirmado](screens/S27.md) | FR-015 | guest / guest | default, closed |
| [S28 — Lembranças guardadas](screens/S28.md) | FR-017, FR-018 | owner / owner | default, loading, error, opening |
| [S29 — Galeria liberada](screens/S29.md) | FR-018, FR-019, FR-020 | owner / owner | default, loading, empty, error, processing, unavailable, blocked |
| [S30 — Visualização em sequência](screens/S30.md) | FR-019, FR-020 | owner / viewer | default, photo, video, audio, text, processing, unavailable, expired-url, blocked |
| [S31 — Convite por e-mail](screens/S31.md) | FR-004, FR-021 | email / email | default, failed, expired |
| [S32 — Código e confirmação por e-mail](screens/S32.md) | FR-001, FR-002, FR-021 + HLD | email / email | default, expired, failed |
| [S33 — Lembranças disponíveis por e-mail](screens/S33.md) | FR-017, FR-018, FR-021 | email / email | default, failed |
| [S34 — Alerta técnico por e-mail](screens/S34.md) | FR-020, FR-022 | operations / email | default, failed |
| [S35 — Indisponibilidade e suporte](screens/S35.md) | FR-022 e RNF | shared / status | default, offline, not-found, forbidden, error, rate-limit, maintenance |
| [D01 — Sair ou descartar rascunho](screens/D01.md) | FR-013 | shared / dialog | default |
| [D02 — Alterar para abertura futura](screens/D02.md) | FR-017, FR-018 | owner / dialog | default, error, pending |
| [D03 — Sair da conta](screens/D03.md) | FR-001 | account / dialog | default, pending, error |

## Cobertura por requisito

| Requisito | Superfícies |
| --- | --- |
| FR-001 | S01, S02, S03, S04, S08, S32, D03 |
| FR-002 | S04, S05, S06, S07, S32 |
| FR-003 | S01, S09, S10, S11, S12 |
| FR-004 | S13, S14, S31 |
| FR-005 | S15, S16 |
| FR-006 | S16, S26 |
| FR-007 | S17, S25 |
| FR-008 | S18, S20, S21, S22, S23 |
| FR-009 | S19, S20, S21, S26 |
| FR-010 | S20, S21, S22 |
| FR-011 | S12, S18, S19, S20, S21, S22, S23, S24 |
| FR-012 | S23, S24 |
| FR-013 | S25, D01 |
| FR-014 | S25, S26 |
| FR-015 | S27 |
| FR-016 | S11, S12, S13, S14 |
| FR-017 | S10, S11, S28, S33, D02 |
| FR-018 | S28, S29, S33, D02 |
| FR-019 | S29, S30 |
| FR-020 | S29, S30, S34 |
| FR-021 | S31, S32, S33 |
| FR-022 | S34, S35 |

Os estados técnicos dos FR-020/022 são mensagens/player/alertas, sem inventar painel de dados de lembranças. A revisão de páginas inferidas e requisitos não especificados está em Q06. Nenhuma aprovação foi presumida.
