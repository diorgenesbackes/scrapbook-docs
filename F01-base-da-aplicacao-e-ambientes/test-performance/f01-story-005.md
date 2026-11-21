# F01-STORY-005 · Carga local da fundação

O k6 v2.3.0 exercitou o candidato integrado de `dev` no Docker Compose descartável, em loopback. O gerador usou quatro GETs por iteração, em `/`, `/ui-health`, `/health` e `/health/ready`, com timeout de três segundos e pausa de um segundo. Houve aquecimento de 10 s com 1 usuário virtual, seguido de três fases de 20 s com 1, 5 e 20 usuários virtuais. A validação exigiu `200` e corpo conforme o contrato de cada rota, sem falhas HTTP.

Foram 2.120 requisições, 530 iterações completas, 4.240 verificações aprovadas e zero falhas HTTP. A prontidão retornou `200` depois da carga. O Docker disponibilizou 12 CPUs e cerca de 7,75 GiB de RAM. O perfil foi executado uma vez, com as rotas saudáveis.

| Rota, fase de 20 usuários virtuais | Amostras | p50 | p95 | p99 |
| --- | --- | --- | --- | --- |
| `/` | 400 | 4,25 ms | 6,52 ms | 8,09 ms |
| `/ui-health` | 400 | 2,25 ms | 4,77 ms | 6,54 ms |
| `/health` | 400 | 3,83 ms | 6,21 ms | 6,57 ms |
| `/health/ready` | 400 | 4,51 ms | 9,99 ms | 11,18 ms |

O resultado aprova a regressão local do shell e das sondas. Teto de escalabilidade, política de rastreamento, latência de rede e reação do WAF exigem medição no ambiente autorizado. [Validação funcional](../test-funcional/f01-story-005.md) · [Preparação de produção](../dev-infra/f01-story-005-producao.md)
