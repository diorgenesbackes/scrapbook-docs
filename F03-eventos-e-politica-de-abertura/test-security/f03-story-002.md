# F03-STORY-002 · Segurança local

| Perspectiva | Controles confirmados |
| --- | --- |
| White box | A consulta com bloqueio aplica o vínculo antes de materializar a linha. A Service compara versão e trava e grava a auditoria na transação da alteração. SQL parametrizado e diagnóstico sanitizado. |
| Grey box | Duas identidades sintéticas no PostgreSQL descartável com privilégio de aplicação. A conta inicialmente alheia foi recusada e ganhou acesso somente depois de receber o segundo vínculo pela fixture. A auditoria foi lida por uma identidade administrativa do laboratório. |
| Black box guiada | Sem identidade, origem inválida, conta alheia e tentativa depois da trava retornaram `401`, `403`, `404` e `409`, respectivamente. As recusas preservaram os registros de auditoria esperados. |

A autenticação real e o aceite remoto dependem do ambiente integrado. [Aceite funcional](../test-funcional/f03-story-002.md)
