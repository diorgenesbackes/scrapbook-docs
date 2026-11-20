# Ensaio de carga local da F01-STORY-002

**Resultado em 19/11/2026.** A fundação respondeu a 80 requisições/s no patamar de 20 VUs, com zero erros e zero timeouts. O p95 de prontidão nesse patamar foi `10,506 ms`. O alvo permaneceu pronto após a carga. [Perfil](1-perfil.md)

## Alvo e ferramenta

A execução usou a `dev` local da API `550872b`, UI `024c436`, infraestrutura `bdf7347`, testes `9c9a15c` e Liquibase `405538d`. A stack descartável teve uma réplica por componente, PostgreSQL 18.6 e Liquibase 4.33.0. O ambiente Docker Desktop usou Docker 29.8.2 e Compose 5.6.0, com 12 CPUs e `7,748 GiB` de RAM na VM. O k6 `2.3.0` rodou no macOS ARM64 do mesmo computador, via HTTP em loopback e com logs ativos. O banco continha a instalação de fundação e as chamadas foram anônimas. [Compose](../../../../scrapbook-infrastructure/compose.yaml)

O roteiro temporário usou `constant-vus`, GET sequenciais, pausa de 1 s e dois checks por resposta. O SHA-256 do script foi `8a471dd575dee2e11ccf082a012da918df8ad83861c50a5d36f06dc4a579d5e9`. O comando `k6 inspect` inicial recusou a configuração por ausência do alvo no ambiente do script. A inspeção com `--env` passou antes de emitir tráfego. O runner conferiu prontidão `200` antes e depois do ensaio e encerrou os recursos descartáveis. [Executor k6](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/constant-vus/)

## Volume e qualidade

| Fase | VUs | Janela | Iterações | Requests | Média sobre a janela | Erros HTTP / timeouts |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| Aquecimento | 1 | 10 s | 10 | 40 | 4 req/s | 0 / 0 |
| Referência | 1 | 20 s | 20 | 80 | 4 req/s | 0 / 0 |
| Carga leve | 5 | 20 s | 100 | 400 | 20 req/s | 0 / 0 |
| Carga local | 20 | 20 s | 400 | 1.600 | 80 req/s | 0 / 0 |

O pico observado foi de 20 VUs. O ensaio terminou com código `0`, 2.120 respostas HTTP `200`, 4.240 checks aprovados e zero iterações descartadas. O tempo total observado pelo runner com amostragem foi `84,94 s`. O modelo é fechado e a média por janela descreve o volume efetivamente entregue.

## Latência por rota

Valores em milissegundos de `http_req_duration`, que soma envio, espera e recepção. Percentis calculados por posição `ceil(p × n)` na amostra ordenada. O aquecimento foi excluído desta tabela desde o planejamento. Com `n=20`, o p99 coincide com a máxima. [Saída JSON do k6](https://grafana.com/docs/k6/latest/results-output/real-time/json/)

| VUs | Rota | n | p50 | p95 | p99 | Máxima |
| ---: | --- | ---: | ---: | ---: | ---: | ---: |
| 1 | `/` | 20 | 2,031 | 2,267 | 2,349 | 2,349 |
| 1 | `/ui-health` | 20 | 0,863 | 1,332 | 1,378 | 1,378 |
| 1 | `/health` | 20 | 2,291 | 3,042 | 4,588 | 4,588 |
| 1 | `/health/ready` | 20 | 4,909 | 6,249 | 6,992 | 6,992 |
| 5 | `/` | 100 | 2,210 | 3,855 | 4,384 | 4,778 |
| 5 | `/ui-health` | 100 | 1,192 | 1,834 | 2,155 | 2,444 |
| 5 | `/health` | 100 | 2,536 | 3,276 | 4,242 | 4,253 |
| 5 | `/health/ready` | 100 | 4,129 | 12,660 | 16,703 | 17,210 |
| 20 | `/` | 400 | 4,236 | 6,687 | 7,597 | 8,655 |
| 20 | `/ui-health` | 400 | 2,356 | 8,211 | 8,818 | 9,214 |
| 20 | `/health` | 400 | 3,675 | 5,629 | 6,559 | 6,695 |
| 20 | `/health/ready` | 400 | 4,443 | 10,506 | 14,062 | 14,408 |

A prontidão teve o maior p95 entre as rotas no patamar de 20 VUs. O p95 a 5 VUs foi maior que a 20 VUs nesta rodada. A diferença é uma observação de execução única.

## Recursos observados

Os picos abaixo vêm de 15 amostras discretas durante o perfil. O percentual de CPU do Docker segue a unidade do `docker stats`. O RSS do k6 vem do `ps` no host.

| Componente | Pico de CPU observado | Memória máxima observada |
| --- | ---: | ---: |
| API | 17,66% | 228,6 MiB |
| Nginx | 3,69% | 21,71 MiB |
| PostgreSQL | 4,60% | 160,6 MiB |
| Gerador k6 | 3,3% | 49,2 MiB de RSS |

## Conclusão por objetivo

| Objetivo | Resultado |
| --- | --- |
| Respostas e prontidão válidas na carga local especificada | Atendido neste perfil. As quatro rotas completaram todos os checks sem timeouts. |
| Latência das operações comuns da API no perfil do PRD | Pendente para as jornadas de negócio. Estes percentis medem rotas operacionais e servem de linha de base local. |
| 200 uploads simultâneos, preparo de mídia e página pública em 4G | Fora do alcance desta execução. O GET da página inicial mediu a resposta HTML. |

O escopo termina em 20 VUs e 20 segundos no patamar final. O teste estabelece comportamento local nesse ponto de carga. [Metas do PRD](../../../PRD.md#4-objetivos)
