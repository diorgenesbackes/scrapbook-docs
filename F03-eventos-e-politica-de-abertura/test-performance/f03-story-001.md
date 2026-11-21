# F03-STORY-001 · Carga local de criação e listagem

Quatro usuários virtuais operaram com duas contas sintéticas por dez segundos em uma stack Docker descartável. Cada iteração criou um evento com chave própria e pediu uma página de até dez eventos. O perfil esperava `201` na escrita, `200` na consulta e ausência de falhas. A referência local de p95 foi 500 ms, conforme as operações comuns do HLD.

| Operação | Requisições | p95 | Falhas HTTP |
| --- | ---: | ---: | ---: |
| Criação | 80 | 14,38 ms | 0 |
| Listagem | 80 | 3,57 ms | 0 |

A versão integrada em `dev` recebeu 160 requisições em loopback, com vazão agregada de 15,55 requisições por segundo. O perfil cobre a regressão local de criação e consulta. Metas de capacidade em AWS e carga de mídia seguem os perfis do produto. [HLD](../../HLD.md#considerações-de-escalabilidade-e-disponibilidade)
