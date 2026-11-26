# F09-STORY-001 — Aceite funcional

O ensaio de 26/11/2026 usou a API `1ee9d816b2339ed4fff79f6d209462f696e03fbe`, UI `57f4e63`, testes `7138d3d`, infraestrutura `56759f3` e Liquibase `a9be70d`. A validação executou em PostgreSQL descartável com SNS e SQS emulados. [E2E e integração HTTP](../test-e2e/f09-story-001.md)

| Critério | Evidência |
| --- | --- |
| Confirmação aciona o fluxo | No ensaio de integração, texto e foto confirmados acionaram a trava e a mídia rejeitada ficou fora do fluxo. No ensaio de rollback, o recibo F08 seguiu disponível depois do commit da contribuição. |
| Atomicidade e recuperação | Uma restrição temporária do PostgreSQL recusou a auditoria em um evento de teste. A transação preservou os campos estruturais, a versão, o convite e o segredo. Auditoria e deduplicação ficaram sem gravação. O fato F08 continuou publicado e outro evento travou durante a falha. |
| Efeito único | Após retirar a restrição temporária, a reentrega natural aplicou uma trava, invalidou um convite, preservou o hash e os vínculos e gravou uma auditoria correlacionada e uma deduplicação. A fila drenou. |
| Edição e abertura | Uma alteração autorizada de abertura enquanto o primeiro fato aguardava continuou vigente após a trava. Escritas estruturais e de convite receberam `409` após consulta da versão atual. |
| Relógio da F03 | Com relógio controlado em uma cópia descartável, a política futura permaneceu flexível até 11h59min59,999s e travou às 12h de Brasília. A política imediata permitiu uma única transição para data futura ao meio-dia e continuou travada às 13h. |
| Acesso | Titular vinculado recebeu `200` com `no-store`. Consulta anônima recebeu `401` e conta autenticada alheia recebeu `404`. |

O ensaio de relógio terminou em 46,891 s. A comparação dos inventários de Docker confirmou a limpeza dos recursos de rollback e abertura. A primeira checagem defensiva, executada na mesma rodada de rollback, encontrou uma asserção de cache excessivamente restritiva. Os resultados funcionais anteriores à asserção foram preservados. [Verificação defensiva](../test-security/f09-story-001.md)
