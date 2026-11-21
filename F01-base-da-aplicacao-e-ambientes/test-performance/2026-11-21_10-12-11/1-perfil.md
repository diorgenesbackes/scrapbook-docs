# F01-STORY-004 · Perfil local de carga e recuperação

Medir a disponibilidade da página inicial e a prontidão da API no proxy da stack. Usar Docker Compose em laboratório descartável, portas aleatórias em loopback, um registro sintético e k6 com modelo aberto de taxa constante. [Referência do executor k6](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/constant-arrival-rate/)

- Aquecer com a verificação de prontidão. Fazer uma fase saudável de 15 segundos e uma de recuperação de 40 segundos, ambas a 20 iterações por segundo, com dois GETs por iteração. Limitar a 40 VUs com timeout de quatro segundos por requisição e até cinco segundos para terminar as iterações em andamento.
- Interromper somente o PostgreSQL descartável durante a segunda fase. Confirmar prontidão `503` e vitalidade `200`. Manter alguns segundos de observação e iniciar o banco com a carga em andamento. Confirmar prontidão `200` e a preservação do registro.
- Aprovar o perfil local com página inicial `200` em ambas as fases, prontidão coerente com o estado do banco, zero descartes, zero falhas inesperadas e retorno antes do encerramento da fase. Medir p50, p95 e p99 por operação e registrar o tempo observado entre o início da retomada e a primeira prontidão confirmada.

O perfil cobre rotas operacionais que existem na F01. O cronômetro do laboratório não mede o tempo de recuperação em homologação, e as metas de upload/processamento do PRD se aplicam às features que entregam essas jornadas. [Critérios do produto](../../../PRD.md#9-critérios-de-aceitação)
