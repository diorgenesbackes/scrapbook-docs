# Revisão técnica da F08-STORY-004

A revisão independente iniciou no serviço `4a8ed35`, na interface `d166ef7`, no changeset `e2c9364` e no template de infraestrutura `b042055`. O build do serviço passou sem avisos. Os checks de interface e SQL passaram no escopo revisado. [Cartão](https://trello.com/c/UR7dLziQ) · [Spec](../spec.md)

| Achado | Correção | Verificação |
| --- | --- | --- |
| Um cabeçalho positivo podia substituir a duração observada nos pacotes. | Inspecionar a faixa reproduzida dos streams de vídeo e áudio, com descarte de pré-rolagem e ajuste do padding informado pelo codec. | Limites inclusivos em unitários, mais vídeos sintéticos no ensaio HTTP. |
| Rotação cardinal negativa podia invalidar o commit no PostgreSQL. | Normalizar para 0, 90, 180 ou 270 antes de persistir o resultado. | MOV em retrato no ensaio local. |
| Um abort multipart com falha podia impedir o avanço de todos os demais. | Reservar cada trabalho de limpeza com lease, verificar partes pelo UploadId e reagendar somente o trabalho afetado. | Cinco novos testes unitários, incluindo dois trabalhos concorrentes e partes residuais. |
| O espaço temporário local de 16 MiB interrompia a validação dos vídeos multipart. | Permitir até 512 MiB no temporário da API no Compose. | Evidência no runner de vídeo e no PR do Compose. |

O reteste confirmou o comparador inclusivo para o evento de criação e o fencing do job. O último build local da branch de serviço passou com 253 testes e sem warnings. O ajuste de formatação incidiu nos arquivos alterados que apresentaram desvios de indentação. [Serviço](../dev-back/f08-story-004.md) · [Infraestrutura](../dev-infra/f08-story-004.md)

A revisão focal de `cbc1225` conferiu a atualização do FFmpeg 9.1, os digests e a assinatura oficial, além do encerramento de tentativas quando o subprocesso falha de modo determinístico. O build Linux ARM64 e a inspeção local dos formatos passaram. O check de PR passou e o serviço foi integrado em dev. [PR do serviço](https://github.com/diorgenesbackes/scrapbook-service/pull/7) · [Verificação defensiva](../test-security/f08-story-004-dev.md)
