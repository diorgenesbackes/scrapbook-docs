# Backend da F08-STORY-003

A API admite até dez fotos de um momento em uma tentativa. A posição consecutiva entra no manifesto canônico em ordem. O banco guarda uma chave temporária e uma chave de original por foto. A autorização vincula evento, tentativa, credencial e arquivo esperado. Uma versão aceita fica fixa e sai da lista de posições pendentes.

A prova de recebimento é gravada sob lock da sessão. A validação começa quando todas as posições têm uma versão tempestiva. A data de recebimento do conjunto é o maior instante entre essas provas, com 13h de Brasília como limite inclusivo. Após o corte, a consulta indica reconciliação enquanto há versões por apurar.

O serviço valida todas as posições antes de copiar originais. Uma rejeição registra o motivo de cada arquivo afetado. A confirmação grava a sessão, o registro, os arquivos, as referências para originais e os fatos da Outbox na mesma transação. O worker reaproveita uma cópia conferida ao repetir uma tentativa após falha de commit. A limpeza usa as versões temporárias elegíveis da própria tentativa. O esquema da F08-STORY-002 já comporta o fluxo. [Especificação](../spec.md) · [Verificação local](../test-funcional/2026-11-25_20-40-42/2-execucao.md)
