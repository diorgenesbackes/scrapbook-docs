# Reutilização da identificação no mesmo evento

A S17 mostra o nome já confirmado e oferece Continuar ou Editar nome. A edição começa com o nome preenchido e a caixa de concordância desmarcada. A nova identificação só é disponibilizada após uma confirmação explícita. [F06-STORY-002](https://trello.com/c/lW6dEG0h) · [S17](../../design-system/screens/S17.md) · [Primeira identificação](f06-story-001.md)

## Estado e persistência

O `GuestIdentificationStep` monta um fluxo por código público e versão do aviso. O `localStorage` guarda o registro identificado pelo código do evento. A chave e o formato compartilhados independem da categoria. O registro contém somente `eventCode`, `guestName`, `noticeCode` e `acceptedAt`. A leitura exige o mesmo evento e aviso, nome sem espaços nas extremidades e instante UTC no formato usado pelo formulário. Uma versão diferente do aviso volta à confirmação explícita. A validação definitiva do aviso e do evento no envio pertence à F08.

Continuar no estado de retorno relê o registro. Se ele tiver sido removido ou se tornado ilegível, a tela mostra a orientação do PRD e solicita novo nome e nova concordância. Quando outra aba tiver salvo uma identificação atualizada, a tela apresenta esse nome para o convidado confirmar. A orientação também aparece quando nenhum registro está disponível na primeira leitura.

O callback entrega `GuestIdentificationConfirmation`, com `identification` no contrato da primeira história e `remembered` indicando se o navegador guardou a confirmação. Se a gravação falhar, o consumidor ainda recebe a confirmação da página atual, e a prévia explica o alcance dessa confirmação. A composição mantém o rascunho em memória fora da etapa de identificação. [PRD](../../PRD.md)

## Validação local

O Compose local com diagnóstico em loopback oferece as prévias `/e/{code}/identificacao/depoimento` e `/e/{code}/identificacao/momento` em eventos abertos. Ambas usam o mesmo componente e um campo de rascunho sintético para verificar a recuperação. O Playwright prepara os eventos e o aviso usando `python3 -B guest_identification_stack.py` em `scrapbook-tests`.

A integração das entradas de composição continua na F07. A confirmação local é autodeclarada e o servidor da F08 será responsável pela validação no envio. O aviso sintético permanece restrito à prévia. [Decisão Q03](../../design-system/decisions.md)
