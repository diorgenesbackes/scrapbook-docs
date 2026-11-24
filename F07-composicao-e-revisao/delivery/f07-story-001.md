# Entrega da F07-STORY-001

A entrada Deixe um depoimento funciona durante a janela da F05. O convidado pode escrever, revisar, editar e descartar um texto. A validação da identificação usa o aviso sintético em Docker local. O contrato para a F08 registra valores escalares Unicode como unidade de contagem a confirmar na decisão Q08. [Card](https://trello.com/c/3Mt5qAFJ) · [Contrato](../dev-front/f07-story-001.md)

| Repositório | Candidato testado em `dev` | Promoção em `hlg` |
| --- | --- | --- |
| `scrapbook-ui` | `43c3862273cc219dc4382d8232a2bd47b78af953` | `3508ff11f9b7e0117924e2a3dbe2b186bbb6e3d6` |
| `scrapbook-tests` | `142c9fce0e3c903062b5732be9d8836def502dc7` | `37185eaba13bb537feb209282b571bf9d03ad5d4` |
| `scrapbook-docs` | Contrato e evidências da história | Promoção seletiva do mesmo conjunto de documentos. |

A comparação das árvores completas confirmou a equivalência de UI e testes entre os candidatos de `dev` e `hlg`. Passaram 82 unitários do frontend, 125 do backend, 20 cenários de navegador da história e 15 regressões. Em 4G emulado, o editor ficou disponível em até 1.828 ms. O ensaio de segurança confirmou isolamento por evento, ausência de escrita HTTP e zero contribuições gravadas. [Revisão](../code-review/f07-story-001.md) · [Navegador](../test-e2e/f07-story-001.md) · [Aceite](../test-funcional/2026-11-24_13-48-27/2-execucao.md) · [Desempenho](../test-performance/f07-story-001.md) · [Segurança](../test-security/f07-story-001.md)

## Publicação e aceite em homologação

GitHub Actions falhou em Reservar a versão. Os jobs de build e publicação foram pulados. [Execução](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45441785785) · [Job de reserva](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45441785785/job/138614601774)

Após publicar o candidato, conferir o SHA em `release.json`, o ingresso da página pública e o acesso direto antes, durante e depois da janela. Conferir edição, revisão e descarte em homologação. A esteira de teste com aviso sintético e identificação fica no Docker local. A F08 integrará Enviar ao fluxo de admissão e confirmação. [Runner](../test-e2e/f07-story-001.md) · [Q03 e Q08](../../design-system/decisions.md)
