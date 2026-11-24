# F04-STORY-001 · Entrega do candidato

O responsável pode emitir, copiar, reexibir e substituir o convite de 24 horas para o segundo responsável. A entrada pelo QR apresenta uma orientação neutra. A prévia autenticada e o aceite fazem parte da segunda story. [Card de emissão](https://trello.com/c/pObm14YS) · [Card de aceite](https://trello.com/c/QW1wB9y2)

| Repositório | Integração em `dev` | Promoção em `hlg` |
| --- | --- | --- |
| Serviço | `ab7a8723c20dc7cc3627756e108c3bc4237d276f` | `5462198f1c410186dccb783249ede5b1df3363ae` |
| Interface | `a61ebca24daa95b68280dc18ef22adcbe223e5eb` | `7a9c12bd84d94b5e966a7ee29133bd17ec5e0264` |
| Testes | `376b774c4b6d726752299eb1107968146bc117ed` | `1d2e2a2950f8f788fcc5c255e7aa05375e98702c` |
| Documentação | `6c51cad505f3e94a20ee03abc44f6cfac06292f1` | `fa82faee22f8300ac8d04cab49eb7e9d3a6cd6da` |
| Infraestrutura | `f555467a4a39dbf6187d16bef272525e54527351` | `69c0e61dd602c528b0caf0a95c2d80163b838491` |

O push foi confirmado nos cinco repositórios. A promoção seletiva preservou a ancestralidade de `hlg`. Serviço, interface e testes têm a mesma árvore validada em `dev`. A F03, o componente de QR da F05 e o schema de convites já existiam nas bases.

Passaram 116 unitários do serviço, 47 da interface, 5 cenários de navegador da F04, 25 regressões da F03 e 15 da F05. Os ensaios HTTP, PostgreSQL e de segurança confirmaram autorização, transação, repetição e concorrência. Em Docker local, a versão integrada mediu p95 de 15,54 ms para consulta e 29,20 ms para emissão. Os 13 itens do card foram verificados e o card passou para To Deploy. [Revisão](../code-review/f04-story-001.md) · [Aceite](../test-funcional/f04-story-001.md) · [E2E](../test-e2e/f04-story-001.md) · [Desempenho](../test-performance/f04-story-001.md) · [Segurança](../test-security/f04-story-001.md)

## Publicação em homologação

Os workflows da API e da interface pararam em “Reservar a versão”. Os dois logs pedem `RELEASE_SERIES` no formato `MAJOR.MINOR`. As etapas de publicação foram puladas. O aceite em homologação fica pendente. [Workflow da API](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/45297278134) · [Workflow da interface](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45297287962)

## Roteiro de aceite em homologação

1. Confirmar os SHAs publicados, a origem HTTPS da interface e a configuração isolada e persistente de proteção dos convites na API do ambiente.
2. Com a identidade de teste prevista para homologação, confirmar emissão, cópia, leitura do QR e reexibição após recarga. Verificar prazo, substituição, lotação e trava usando eventos de teste.
3. Conferir os headers na entrada publicada e a ausência de tokens utilizáveis na telemetria. Completar o aceite autenticado com a F04-STORY-002 e a integração com a F02. [Spec](../spec.md)
