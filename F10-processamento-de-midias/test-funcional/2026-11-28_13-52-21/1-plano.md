# Plano de validação funcional da F10-STORY-004

Validação independente do candidato integrado em `dev`, em 28/11/2026, no fuso America/Sao_Paulo. O procedimento usa somente recursos descartáveis com portas em loopback. [Cartão](https://trello.com/c/dLVALXU9) · [Spec F10](../../spec.md) · [Plano F10](../../plan.md) · [PRD](../../../PRD.md#f10-processamento-de-mídias-2)

| Critério | Cenário e evidência esperada |
| --- | --- |
| CA1 | Um operador simula e aplica a varredura paginada. Uma confirmação com Outbox e outra com comprovante F08 preservam identidade, instante, original e recibo. Um caso inconclusivo fica sem trabalho e auditado. Uma foto nova progride enquanto um trabalho histórico está bloqueado antes da promoção. |
| CA2 | Queda de duas réplicas entre gravação no S3 e commit leva à recuperação do lease e a uma única promoção. Perda de resposta e callbacks antecipados, atrasados ou ausentes convergem na tentativa em andamento. Redrive aplica o fato original, preserva a deduplicação após o horizonte e recusa reconstituir efeito sem prova. |
| CA3 | Operador simula e reabre uma falha definitiva corrigida, com seleção por evento, arquivo, perfil e ciclo e com incidente e revisão. Repetição e seleção fora do escopo são rejeitadas. Verificar a identidade do trabalho, os números das tentativas, os fatos definitivos por ciclo e a integridade da confirmação. |
| CA4 | Ler o contrato interno da F11 para um evento, registro e arquivo. Conferir representação dos estados, referências versionadas do perfil ativo e ausência de variantes parciais. Simular e aplicar limpeza depois de 24 horas da tentativa encerrada, preservando versões sem vínculo, originais, recibos e variantes prontas. |

A suíte de recuperação opera o CLI real com role PostgreSQL exclusiva. As regressões de foto, áudio e vídeo percorrem as jornadas da F08 no Chromium, a API, PostgreSQL e Moto. A fixture de contrato do MediaConvert aceita perfis e publica artefatos sintéticos que o candidato precisa verificar. O alcance inclui o orquestrador e as regras locais. MediaConvert, EventBridge e IAM reais são ensaios de homologação. A experiência de galeria e os alertas entregues pela F13 são integrações posteriores.
