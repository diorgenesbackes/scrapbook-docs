# Aceite local da seleção de fotos

A execução integrada em `dev` passou nos quatro critérios. [Ambiente e perfis](../../test-e2e/f07-story-002.md)

| Critério | Observado | Resultado |
| --- | --- | --- |
| CA1 | As duas rotas funcionaram na janela aberta, e o ingresso direto recusou janela fechada. O depoimento manteve uma foto. Momento reteve dez válidas e indicou separadamente a foto excedente. O unitário aceitou 20.000.000 bytes e a suíte de navegador recusou um byte a mais. | Aprovado |
| CA2 | JPEG e PNG exibiram prévia. Vazio, formato incompatível e PNG corrompido receberam mensagens distintas. A amostra HEIC exibiu prévia nos perfis com suporte e recuperação nos demais. | Aprovado no recorte |
| CA3 | A troca manteve as posições e a remoção manteve a ordem restante. A legenda única atingiu 500 caracteres e avisou aos 100 restantes. Os unitários validaram Files originais, ordem e legenda no contrato. | Aprovado |
| CA4 | Permanecer conservou as fotos e devolveu foco. Troca, remoção e descarte revogaram URLs. A suíte observou zero escritas HTTP a partir do fluxo de revisão e zero registros no banco de teste. | Aprovado |

A F08 vinculará a ação de enviar à validação e ao recebimento integral. [Plano](1-plano.md) · [Contrato](../../dev-front/f07-story-002.md)
