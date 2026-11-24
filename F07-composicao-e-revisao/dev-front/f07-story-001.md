# Composição e revisão do depoimento em texto

O convidado pode abrir a composição na página pública durante a janela, escolher texto, escrever, editar e revisar. A rota direta `/e/{code}/depoimento` resolve o mesmo código e reconsulta a janela fornecida pela F05. Após o encerramento, a interface mantém o rascunho que já estava em memória e impede a progressão até o envio. [Trello](https://trello.com/c/3Mt5qAFJ) · [PRD](../../PRD.md#f07-composição-e-revisão) · [Janela da F05](../../F05-pagina-publica-qr-code-e-prazos/spec.md#contrato-de-janela-para-f07-e-f08)

O editor valida texto em branco e até 5.000 caracteres, contando espaços, pontuação e quebras de linha. Mostra o contador a partir de 500 restantes. Colagem excedente aceita a parte que cabe sem remover o trecho fora da seleção. O rascunho reside no estado da página. A saída pelos links controlados e a ação Descartar abrem o diálogo D01 com foco em Permanecer. O aviso ao recarregar ou fechar depende do navegador. [S23](../../design-system/screens/S23.md) · [D01](../../design-system/screens/D01.md)

## Contrato para a F08

| Campo | Valor |
| --- | --- |
| `eventCode` | Código resolvido no contexto público da F05. |
| `category` | `testimonial`. |
| `format` | `text`. |
| `text` | Texto original validado, incluindo espaços úteis e quebras de linha. |
| `identification` | Snapshot confirmado pela F06 para o mesmo `eventCode`, acrescentado após a confirmação. |

O contrato proposto para a F08 conta valores escalares Unicode, conforme o protótipo citado na Q08. Um caractere suplementar conta uma vez, e marcas combinantes contam individualmente. A mesma unidade deverá ser usada na legenda de 500 caracteres e na validação autoritativa do serviço. A Q08 permanece aberta para o aceite do contrato definitivo. [Decisão Q08](../../design-system/decisions.md)

A revisão reutiliza a identificação da F06 no Docker local com aviso sintético. O aviso publicado depende da Q03. A ação Enviar está reservada à F08, que validará janela, identificação e conteúdo antes de admitir a tentativa. O fluxo atual não envia nem persiste o rascunho. [S25](../../design-system/screens/S25.md) · [Identificação da F06](../../F06-identificacao-do-convidado/dev-front/f06-story-002.md)
