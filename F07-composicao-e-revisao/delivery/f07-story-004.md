# Entrega da F07-STORY-004

Convidados podem fotografar para um depoimento ou momento, refazer uma foto na mesma posição ou escolher um arquivo do dispositivo. A revisão apresenta a prévia e a legenda. A câmera abre por ação explícita e o JPEG usa até 1.920 px no lado maior. [Card](https://trello.com/c/uxlzqFB7) · [Contrato](../dev-front/f07-story-004.md)

| Repositório | Candidato testado em `dev` | Candidato em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `d05fb32f5db211bfdc91b4091078173e7316d818` | `0f34d6a1e68e47177dcde1c5d9559756f4f0422b` |
| `scrapbook-tests` | `97c2e35f4a779aec8e71fb96bf391901c74fb429` | `1d6f10911dc8ed611be2515488535582b5c38881` |
| `scrapbook-docs` | Contrato e evidências | Promoção dos mesmos documentos. |

As árvores da UI e dos testes coincidiram entre os candidatos. Passaram 104 unitários da UI, 125 do serviço e 85 cenários de navegador. O ensaio adicional em 4G emulado deixou o editor utilizável em até 1.957 ms, encerrou a trilha sintética em até 5 ms e apresentou a prévia em até 25 ms após a captura. A suíte observou zero escritas HTTP e zero contribuições gravadas. [Revisão](../code-review/f07-story-004.md) · [Navegador](../test-e2e/f07-story-004.md) · [Aceite](../test-funcional/2026-11-24_19-05-34/2-execucao.md) · [Desempenho](../test-performance/f07-story-004.md) · [Segurança](../test-security/f07-story-004.md)

## Publicação e aceite em homologação

GitHub Actions falhou na etapa Reservar a versão. Os jobs de build e publicação foram pulados. A publicação deste candidato em homologação está pendente. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45498788069) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45498788069/job/138816219329)

Após a publicação, conferir o SHA em `release.json` e o cabeçalho `Permissions-Policy` servido pelo Amplify. Em aparelhos físicos, testar pedido de câmera explícito, acesso recusado com fallback, captura nas duas categorias e liberação da câmera ao concluir ou sair. [Jornada local](../test-e2e/f07-story-004.md)
