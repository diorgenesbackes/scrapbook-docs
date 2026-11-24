# Primeira identificação do convidado

História [F06-STORY-001](https://trello.com/c/RiJ79nyD), requisito [FR-007 e F06](../../PRD.md) e tela [S17](../../design-system/screens/S17.md).

## Contexto e concordância

A S17 recebe o nome do evento e usa o código público como chave de contexto, conforme o [contrato da F05](../../F05-pagina-publica-qr-code-e-prazos/spec.md). A página usa a consulta anônima da F05 e oferece a identificação quando o servidor informa que a janela está aberta. A troca do código remonta a jornada e exige uma nova confirmação.

O convidado informa nome ou apelido autodeclarado, lê o aviso e marca a concordância. Alterar o nome depois de marcar a caixa exige nova concordância. O formulário remove espaços nas extremidades, preserva o conteúdo digitado durante a validação e move o foco para o primeiro campo inválido.

## Contrato de integração

O mesmo formulário pode ser usado em depoimento e momento. A ação Continuar entrega estes campos ao fluxo que montou o formulário.

| Campo | Significado |
| --- | --- |
| `eventCode` | Código público resolvido pela F05. A identificação pertence a esse evento. |
| `guestName` | Nome ou apelido autodeclarado, sem espaços nas extremidades. |
| `noticeCode` | Código da versão do texto exibido, conforme a [modelagem de avisos](../../db/DB-Modelo-Fisico.md#recordsconsent_notices). |
| `acceptedAt` | Instante ISO UTC da confirmação capturado no navegador. A F08 definirá a evidência autoritativa no servidor e validará os dados no envio. |

O formulário recebe a versão e o texto do aviso como entrada. A fonte prevista para o aviso oficial é a versão vigente e imutável do modelo `consent_notices`. A aprovação de privacidade e a integração da fonte oficial antecedem a disponibilidade pública do fluxo. [Decisão Q03](../../design-system/decisions.md)

## Verificação local

O modo de diagnóstico do Compose local habilita `/e/{code}/identificacao` em loopback. Use o código de um evento cuja janela de contribuição esteja aberta. A prévia usa o aviso sintético `f06-local-v1` e sinaliza a conclusão no navegador. A validação automatizada provisiona dois eventos e um relógio controlado com `python3 guest_identification_stack.py` em `scrapbook-tests`.

A F06-STORY-002 implementará a reutilização no navegador. A F07 colocará a S17 nos fluxos de composição e a F08 validará a identificação e confirmará o registro no evento.
