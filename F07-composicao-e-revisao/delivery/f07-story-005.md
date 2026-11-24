# Entrega da F07-STORY-005

Convidados podem gravar vídeo no depoimento ou no momento e áudio no depoimento. Podem parar manualmente. O contador encerra a gravação após 60 s no momento e 300 s no depoimento, e o arquivo válido abre a revisão local. O editor permite refazer, trocar por arquivo, legendar e descartar. [Card](https://trello.com/c/AtKDv6RF) · [Contrato](../dev-front/f07-story-005.md)

| Repositório | Candidato testado em `dev` | Candidato em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `61565658e88cd9510be1c0724f5dd796d1e8db02` | `f40092af18a8940f1cb82804409c8dd2e23731f3` |
| `scrapbook-tests` | `948bc5ad2a57ec6182e506920e48a0b2d4508bd2` | `36985f76d4ba204921de7184050985c4cf9a5f51` |
| `scrapbook-docs` | Contrato e evidências | Promoção dos mesmos documentos. |

As árvores da UI e dos testes coincidiram entre os candidatos. Passaram 118 unitários da UI, 125 do serviço e 103 cenários de navegador. Em 4G emulado, o editor ficou utilizável em 1.987 ms. A gravação de um minuto gerou 59,986 s reproduzíveis e encerrou ambas as trilhas 1 ms após a parada. A verificação defensiva confirmou zero escritas HTTP e zero contribuições persistidas. [Revisão](../code-review/f07-story-005.md) · [Navegador](../test-e2e/f07-story-005.md) · [Aceite](../test-funcional/2026-11-24_19-24-37/2-execucao.md) · [Desempenho](../test-performance/f07-story-005.md) · [Segurança](../test-security/f07-story-005.md)

## Publicação e aceite em homologação

GitHub Actions falhou na etapa Reservar a versão. Os jobs de build e publicação foram pulados. A publicação em homologação segue pendente. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45502434532) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45502434532/job/138828890060)

Após a publicação, conferir o SHA em `release.json` e o cabeçalho `Permissions-Policy`. Em dispositivos físicos, testar permissão de câmera e microfone, seleção de arquivo após recusa, reprodução, refação, parada no limite e liberação de dispositivos. [Matriz de navegador](../test-e2e/f07-story-005.md)
