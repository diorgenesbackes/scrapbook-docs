# Entrega da F07-STORY-003

Depoimento e momento permitem selecionar e revisar vídeo. Depoimento também aceita áudio. O navegador confere formato, tamanho, duração e disponibilidade da prévia. A seleção pode ser trocada ou removida, e a reprodução começa por ação do convidado. Os limites decimais são 500.000.000 bytes para vídeo e 50.000.000 para áudio. [Card](https://trello.com/c/XbEzx9C2) · [Contrato](../dev-front/f07-story-003.md)

| Repositório | Candidato testado em `dev` | Candidato em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `93e4b9c57fb5ce7c5426171d6508199ee41630e4` | `9b76421f526e120dc410ac3c95639cb3c57feea0` |
| `scrapbook-tests` | `45a0ff98348fc10b9809a80c542040244c1ce711` | `272ba3fe506898f7ef0a7e2df96acf247e3ef368` |
| `scrapbook-docs` | Contrato e evidências da história | Promoção dos mesmos documentos. |

As árvores completas da UI e dos testes coincidiram entre os candidatos. Passaram 96 unitários da UI, 125 do serviço e 75 cenários de navegador. No ensaio adicional em 4G emulado, o seletor ficou utilizável em até 1.967 ms e as prévias ficaram disponíveis em até 77 ms após a seleção automatizada das amostras. A verificação de segurança encontrou zero escritas HTTP e zero contribuições gravadas durante as jornadas de revisão. [Revisão](../code-review/f07-story-003.md) · [Navegador](../test-e2e/f07-story-003.md) · [Aceite](../test-funcional/2026-11-24_17-50-00/2-execucao.md) · [Desempenho](../test-performance/f07-story-003.md) · [Segurança](../test-security/f07-story-003.md)

## Publicação e aceite em homologação

GitHub Actions falhou na etapa Reservar a versão. Os jobs de build e publicação foram pulados, e o candidato segue pendente de publicação em homologação. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45485514092) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45485514092/job/138769705257)

Após a publicação, conferir o SHA em `release.json` e a diretiva `media-src` no cabeçalho. Em aparelhos físicos, testar a seleção nas duas categorias, a recuperação de mídia incompatível, a legenda e a interrupção de reprodução ao voltar ou descartar. [Matriz local](../test-e2e/f07-story-003.md)
