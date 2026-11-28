# Revisão técnica da F10-STORY-005

Revisão independente de 28/11/2026, sobre a [especificação](../spec.md), o [cartão](https://trello.com/c/VIdGX9ti) e o [procedimento de ativação](../dev-infra/f10-story-005.md).

| Repositório | SHA de origem reavaliado | Check |
| --- | --- | --- |
| Serviço | `062975160c99e9a33e0a1a21e00b60e33c3f56a1` | Build sem erros ou avisos. 825 testes unitários aprovados, nenhum ignorado. [PR](https://github.com/diorgenesbackes/scrapbook-service/pull/25) |
| Infraestrutura | `0cc73f2798af537ba7fbb85e8d5c2d4163cc9516` | 28 testes Python aprovados. [PR](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/22) |
| Testes | `9287ad609fdf607f0e8d245365252ccae73bf9ed` | Nove testes do cálculo misto e cinco do cálculo histórico aprovados. `node --check` e inspeção k6 aprovados. [PR](https://github.com/diorgenesbackes/scrapbook-tests/pull/25) |
| Documentação | `007f2af19e054231e4b52dcbad2d6f2e55a27582` | Consistência de flags e sinais conferida. [PR](https://github.com/diorgenesbackes/scrapbook-docs/pull/35) |

| Achado | Resolução reavaliada |
| --- | --- |
| P2 · Drenagem incluía trabalhos em execução sem lease vigente. | O snapshot compara os leases com um único instante. Apenas leases vigentes entram em ativos. |
| P2 · A instrução antiga de pausa desligava o processamento. | O procedimento explicita `MediaProcessing__Paused` e a continuidade da reconciliação autorizada. |
| P2 · Tentativa que exige intervenção podia seguir em reconciliação sem notificação. | O template gera alarme independente em `reconciliation/intervention_required`, agregado e sem dimensões. |
| P3 · O runner de homologação exigia uma sobreposição exata de 200 jornadas. | O k6 mantém 200 jornadas e recibos obrigatórios nos thresholds. A sobreposição atingida fica na evidência. |

A reavaliação encerrou os quatro achados. O formato de whitespace nos arquivos alterados passou. A execução global de `dotnet format --severity info` apontou 22 ocorrências de whitespace preexistentes e sete sugestões informativas nos trechos alterados. A revisão não mediu cobertura percentual. A UI de referência `57f4e63` passou lint, build e 188 testes. Os checks de CI dos quatro PRs para `dev` passaram.

## Revisão das correções da validação local

- O teste de pausa usa a API com relógio real. O evento sintético toma o horário do PostgreSQL após o PUT e normaliza o fuso para UTC. A revisão confirmou que as asserções existentes e os dados do objeto são preservados. [PR da pausa](https://github.com/diorgenesbackes/scrapbook-tests/pull/26)
- O coletor da carga mista local usa horário do banco após observar as versões enviadas, com precisão de microssegundos. A revisão confirmou que o legado da F08 conserva `LastModified` e que o modo HTTPS, a contagem e os limites de aceite permanecem iguais. [PR da carga](https://github.com/diorgenesbackes/scrapbook-tests/pull/27)
