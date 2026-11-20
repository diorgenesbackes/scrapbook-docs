# Perfil de carga local da F01-STORY-002

Medir a linha de base das rotas de fundação por meio do proxy da UI, na `dev` local. Usar uma réplica de cada componente em projeto Compose descartável, PostgreSQL preparado pelo Liquibase e identidade sintética desativada. O alvo aceita somente a porta de loopback alocada ao projeto. [Spec da F01](../../spec.md#5-contratos-de-api) · [Metas do PRD](../../../PRD.md#4-objetivos) · [Executor constant-vus](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/constant-vus/)

| Fase | Início | VUs | Janela |
| --- | --- | --- | --- |
| Aquecimento | 0 s | 1 | 10 s, separado dos resultados |
| Referência | 13 s | 1 | 20 s |
| Carga leve | 36 s | 5 | 20 s |
| Carga local | 59 s | 20 | 20 s |

Cada iteração executa em sequência `GET /`, `GET /ui-health`, `GET /health` e `GET /health/ready`, por VU, e aguarda 1 s. O modelo é fechado. Cada rota tem o mesmo peso. As fases têm até 3 s para drenagem, sem sobreposição, e o ensaio dura até 82 s. O teto é 20 VUs e a oferta nominal é 80 requisições/s no último patamar. Cada chamada tem timeout de 3 s. O perfil espera HTTP `200` com o shell em pt-BR ou JSON operacional `up`, conforme a rota. Erros HTTP e falhas de contrato acionam interrupção após a janela inicial de avaliação.

Registrar amostras, latência HTTP p50/p95/p99 e máxima por rota, erros, checks, throughput, iterações e observações de CPU e memória dos contêineres. O p95 de 500 ms do PRD serve apenas como referência de escala, pois esta história oferece rotas operacionais e um shell HTML. A meta de carga com 200 envios, processamento de mídia e prontidão em 4G exige os fluxos correspondentes. [Saída JSON do k6](https://grafana.com/docs/k6/latest/results-output/real-time/json/)
