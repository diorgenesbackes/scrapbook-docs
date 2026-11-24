# Verificação defensiva da identificação lembrada

Em 24/11/2026, os controles foram verificados em Docker descartável com evento, convidado e rascunho sintéticos. [Ambiente e candidato](../test-e2e/f06-story-002.md)

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| White box | A leitura valida evento, versão do aviso, nome e timestamp antes de reconstruir o objeto. Os unitários testam perda do registro, troca de aviso, edição e falhas de leitura e escrita. O JavaScript do build padrão exclui o aviso, a chave local e os controles de diagnóstico. | Aprovado |
| Grey box | As duas categorias do mesmo evento usaram o mesmo registro. A chave de outro evento com conteúdo incompatível e o registro sem instante válido voltaram à confirmação. A stack registrou zero contribuições gravadas. | Aprovado |
| Black box guiado | A navegação anônima mostrou somente dados do evento corrente. O formulário não colocou nome ou concordância na URL, e a confirmação manteve os GETs públicos observados. A verificação da stack não encontrou nomes, versão do aviso ou rascunho sintético na telemetria. | Aprovado |

Sem achados remanescentes nos controles verificados. A F08 valida a identificação autodeclarada no servidor na etapa de envio. A aprovação do aviso e das regras de retenção precede a liberação pública. [Contrato](../dev-front/f06-story-002.md) · [Decisão Q03](../../design-system/decisions.md)
