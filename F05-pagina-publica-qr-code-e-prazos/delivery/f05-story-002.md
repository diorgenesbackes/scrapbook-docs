# F05-STORY-002 · Entrega do candidato

Os responsáveis vinculados podem copiar o endereço canônico, baixar o QR em PNG ou SVG e imprimir a folha de 60 mm. A página dos convidados mostra o mesmo QR em todos os estados da janela. A promoção seletiva preservou as bases de `hlg`. Serviço, interface e testes têm a mesma árvore que a versão validada de `dev`. A F03, a F05-STORY-001 e seu schema já integram as bases. [Card](https://trello.com/c/ZbToV6J7)

| Repositório | Integração em `dev` | Promoção em `hlg` |
| --- | --- | --- |
| Serviço | `ab11a1a8528e968f8ab944dfc8392e07a31aa07f` | `2fe95ee3158bc0abe54c37d325ac8ee794d52656` |
| Interface | `423682fa355afdb9eb261c841a8b4178b55d0d8d` | `72a6dc48f2a9e5c0490a575f0678508821e774ae` |
| Testes | `68421268ba22948471b1a2f4f5bc3bb78e21225d` | `23295b087c720754844b69eebc50b86c106181d6` |
| Documentação | `5d4853d926b5a372c11590a2c9fb571279688373` | `34b02fe690f13058dc5b47618e24fd9c835745ed` |
| Infraestrutura | `c1b8c9f186482efbc741f104ef84210201ea2b49` | `55d541c8846ba93194cf5061b343d2cdedd04f8e` |

O candidato passou em 108 unitários do serviço, 41 da interface, 15 cenários Playwright da F05, 30 regressões da F03, integração HTTP e PostgreSQL e verificações defensivas. A página com QR ficou pronta em 2.150 ms no perfil 4G. As 600 consultas de carga passaram com p95 de 54,03 ms. O T11 ficou aberto para a leitura do QR em papel com Android e iPhone. [Revisão](../code-review/f05-story-002.md) · [Aceite local](../test-funcional/f05-story-002.md) · [E2E](../test-e2e/f05-story-002.md) · [Segurança](../test-security/f05-story-002.md) · [Desempenho](../test-performance/f05-story-002.md)

## Publicação em homologação

Os cinco pushes foram confirmados e o card foi movido para To Deploy. Os workflows da API e da interface falharam em “Reservar a versão”. Os dois logs registraram “Use a série MAJOR.MINOR em RELEASE_SERIES.” A publicação dos dois componentes foi pulada. O aceite em homologação depende da configuração da série de release e de uma execução bem-sucedida dos workflows. [Workflow da API](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/45229946462) · [Workflow da interface](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45229960080) · [Política de versões](../../guidelines/git-flow.md#9-releases-e-artefatos)

## Aceite em homologação

Confirmar os dois SHAs publicados e a origem HTTPS do ambiente. Usar o roteiro de aceite para conferir o alias, os headers publicados, a leitura do papel com Android e iPhone e o download no Safari iOS. O mesmo roteiro inclui a verificação do domínio definitivo antes da tiragem do piloto. [Roteiro de aceite](../test-funcional/f05-story-002.md#roteiro-para-o-domínio-definitivo-e-a-tiragem)
