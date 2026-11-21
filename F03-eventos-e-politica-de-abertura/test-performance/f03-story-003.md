# F03-STORY-003 · Ensaio local de concorrência

k6 2.3.0 no host, com serviços em Docker local e PostgreSQL descartável e dois serviços que representam duas contas vinculadas. Seis usuários virtuais consultaram e editaram o mesmo evento por 10 segundos, com pausa de 200 ms por iteração. O perfil força disputa pela versão e aceita apenas uma gravação consistente ou `EVENT_VERSION_CONFLICT`.

| Métrica | Candidato | Integração em `dev` |
| --- | --- | --- |
| Amostras de edição | 282 em 564 requisições totais | 282 em 564 requisições totais |
| p50 / p95 / máximo do PUT | 5,43 ms / 7,45 ms / 8,57 ms | 5,58 ms / 7,51 ms / 9,05 ms |
| Edições aceitas | 47 | 47 |
| Conflitos esperados | 235 de 282 (83,33%) | 235 de 282 (83,33%) |
| Edições tentadas por segundo | 28,15 | 28,05 |
| Falhas inesperadas | 0 | 0 |
| Meta de p95 de até 500 ms | Atendida no perfil local | Atendida no perfil local |

O perfil usa uma única linha para medir contenção. A taxa de conflitos descreve esse cenário. Os ensaios de regressão de criação, listagem e abertura também passaram.

[Execução e versão](../delivery/f03-story-003.md) · [Critérios da F03](../spec.md)
