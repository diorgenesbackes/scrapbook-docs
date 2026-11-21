# F03-STORY-004 · Aceite funcional local

| Critério | Evidência | Resultado |
| --- | --- | --- |
| CA1 | As duas contas vinculadas consultaram o estado. A terceira recebeu o mesmo 404 do evento inexistente. | Aprovado |
| CA2 | Um milissegundo antes da abertura, o HTTP devolveu fechado com somente os quatro campos documentados, mesmo com um registro confirmado sintético. A página mostrou data e contador acessível. | Aprovado |
| CA3 | À meia-noite, detalhe e consulta mudaram para aberto com zero avisos gravados. Após salvar uma nova política de imediato para futuro, a consulta voltou a fechado e a mudança persistiu no PostgreSQL. | Aprovado |
| CA4 | Com relógio do navegador ajustado para 2041, a página seguiu a referência do servidor. Unitários entregaram uma resposta antiga depois da troca de evento e depois de 401 e confirmaram seu descarte. A fronteira da F11 foi documentada. | Aprovado |

A F02 fornecerá sessões reais. A F11 fornecerá o conteúdo e a F12 encaminhará avisos. A validação atual usa identidades de desenvolvimento e um relógio de teste na cópia descartável do serviço.

[Contrato](../spec.md#estado-de-abertura-e-autorização-da-galeria) · [Validação de navegador](../test-e2e/f03-story-004.md)
