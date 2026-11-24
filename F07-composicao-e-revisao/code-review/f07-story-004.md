# Revisão técnica da F07-STORY-004

A revisão percorreu a F05 até a entrada do editor, a permissão de dispositivo, a conversão em JPEG, a seleção de fotos e o retorno da revisão. O serviço permanece no SHA `76fd9e0`. [Contrato](../dev-front/f07-story-004.md) · [Base de fotos](../dev-front/f07-story-002.md)

- A permissão nasce da ação Fotografar ou Refazer. A captura só habilita após um quadro disponível. A falha de prévia tem recuperação após erro, encerramento da trilha ou timeout.
- O identificador da tentativa impede que autorização ou conversão tardias alterem um rascunho encerrado. A câmera solicitada tardiamente é liberada. A troca de arquivo conserva a foto anterior até que a nova seja validada.
- A política de câmera limita o acesso à própria origem do documento. Os cinco conjuntos de cabeçalhos gerados para o Amplify mantêm microfone e geolocalização bloqueados e `frame-ancestors 'none'`. [Referência do formato](https://docs.aws.amazon.com/amplify/latest/userguide/custom-header-YAML-format.html)

| Check | Resultado |
| --- | --- |
| Oxlint, TypeScript, Vite e Vitest | Exit 0 e 104 unitários aprovados. A saída do compilador e do lint não exibiu avisos. |
| Restore bloqueado, compilação limpa e unitários .NET 10.0.401 | Exit 0, zero avisos e 125 unitários aprovados. |
| Reteste dirigido em Docker | Dez cenários de câmera passaram. A execução avulsa emitiu o aviso de coexistência de `NO_COLOR` e `FORCE_COLOR`. O runner normal remove `NO_COLOR` antes de invocar o Playwright. |
| `git diff --check` e verificação do pacote Amplify | Exit 0. |

O primeiro ensaio de câmera retornou `NotSupportedError` no Chromium headless. Um ensaio isolado confirmou a necessidade dos dois argumentos de dispositivo e interface sintéticos. Após o ajuste do executor, passaram os dez cenários dirigidos. O runner completo também foi ajustado para usar os mesmos dados sintéticos esperados pela regressão da página pública.

Os checks unitários verificaram recuperação, cancelamento, revogação das trilhas e formato do JPEG com as fronteiras de quantidade e tamanho da seleção de fotos. A execução não coletou percentuais de cobertura. Sem achados técnicos remanescentes no escopo revisado.
