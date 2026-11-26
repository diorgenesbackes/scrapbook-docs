# Plano de aceite da F08-STORY-003

Usar dois eventos, fotos e aviso sintéticos, armazenamento versionado e relógio controlado. A candidata parte da `dev` sincronizada e incorpora, nas worktrees da story, os SHAs publicados da F08-STORY-002. Confirmar a ordem de integração das dependências antes dos PRs para `dev`. [Cartão](https://trello.com/c/nvB32cdj) · [Dependência](../../delivery/f08-story-002.md) · [Especificação](../../spec.md)

| Critério | Exercício e evidência |
| --- | --- |
| CA1 | Admitir conjuntos de uma e dez fotos. Recusar onze, chaves repetidas, posições repetidas ou fora da ordem, tamanho acima de 20.000.000 bytes e legenda de 501 escalares. Aceitar 500 escalares. |
| CA2 | Receber parte do conjunto, recuperar o status com a mesma credencial, autorizar apenas posição pendente e comparar os bytes exibidos com os transferidos. Conferir isolamento de evento, tentativa e arquivo. |
| CA3 | Usar o maior instante entre as versões aceitas e admitir conclusão às 13h exatas. Recusar confirmação integral diante de foto ausente, tardia ou inválida e apontar as posições atingidas. |
| CA4 | Preservar originais privados, registrar ordem e legenda e devolver o mesmo recibo após replay. Conferir registro único, fatos por arquivo e limpeza apenas de temporários elegíveis. |

Repetir os fluxos de texto e depoimento de foto na versão integrada. Medir prontidão, progresso e p95 em conjuntos representativos de uma e dez fotos. Registrar as verificações executadas no ambiente emulado e o aceite pendente na AWS, incluindo evidência do S3, CORS e IAM. [Estratégia de validação](../../spec.md#7-estratégia-de-testes)
