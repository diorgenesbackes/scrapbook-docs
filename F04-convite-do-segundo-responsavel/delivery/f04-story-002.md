# F04-STORY-002 · Entrega do candidato

Uma conta ativa e confirmada pode abrir a prévia do convite, aceitar e acessar o mesmo evento em “Meus eventos”. O aceite usa a vaga de segundo responsável e registra consumo e auditoria na mesma transação. O resultado pode ser recuperado pela mesma conta enquanto o vínculo existir. [Card](https://trello.com/c/QW1wB9y2)

| Repositório | Integração em `dev` | Promoção em `hlg` |
| --- | --- | --- |
| Serviço | `76fd9e084afcfb24ccb2521aaed6d61d56190a03` | `94e79419d20dbb6d22be67f2256e0a08132d26b6` |
| Interface | `7d7d08ed3d844628ebb67a0e3d8e15e51e445ccb` | `31ce11006bfb49d9163a64693dba33da8db87eb9` |
| Testes | `e23338fdcb6fcb6cea10f4058c4a956f50a524e0` | `55d090297dc1cba275b41e07bbe1d30bf04ecff4` |
| Documentação | `14d504f066e9b8f4fa3fb54cb347fe64af2159b3` | `f804b776ae2121fcf34bd4a71bc923af72c3c099` |

Os quatro pushes foram conferidos no remoto. A promoção preservou a ancestralidade dos destinos e a árvore validada do serviço, da interface e dos testes. A F03, a emissão da F04 e o schema já estavam nas bases. O card está em To Deploy, com 13 itens concluídos.

Passaram 125 unitários do serviço e 50 da interface, 5 cenários de aceite, 5 de emissão e 25 regressões da F03. O PostgreSQL confirmou a borda do prazo, um único vencedor na disputa, recusa após a trava e rollback quando a auditoria falha. As 240 respostas da carga integrada passaram com p95 máximo de 18,12 ms. [Revisão](../code-review/f04-story-002.md) · [Aceite](../test-funcional/f04-story-002.md) · [E2E](../test-e2e/f04-story-002.md) · [Desempenho](../test-performance/f04-story-002.md) · [Segurança](../test-security/f04-story-002.md)

## Publicação em homologação

Os workflows da API e da interface falharam em “Reservar a versão” e pularam a publicação. O log da API pede `RELEASE_SERIES` no formato `MAJOR.MINOR`. O aceite em homologação aguarda uma publicação bem-sucedida. [Workflow da API](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/45391939909) · [Workflow da interface](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45391953185)

## Roteiro de aceite em homologação

1. Conferir os SHAs publicados, a origem HTTPS da interface e as chaves de proteção de convites do ambiente.
2. Com duas identidades confirmadas de teste, emitir, ler o QR, abrir a prévia e confirmar o aceite. Conferir a listagem e o detalhe da F03. Exercitar expiração, substituição, trava e erro de resposta após um aceite válido.
3. Verificar cabeçalhos e telemetria do ambiente. Executar login, confirmação de e-mail e retomada de convite na integração F02. Executar o encaminhamento por e-mail na integração F12.
