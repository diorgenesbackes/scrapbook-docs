# Revisão da F08-STORY-001

- Application concentra as regras de admissão, prazo, idempotência e confirmação. Infrastructure usa parâmetros tipados, transação no `ScrapBookDbContext` e o lock de evento comum às operações que alteram a sua janela. O primeiro fato fica na mesma transação do texto. [Contrato de backend](../dev-back/f08-story-001.md)
- A API usa o leitor JSON existente, limites de corpo e o mapeamento central de Problem Details. Prova, segredo e fingerprint usam HMACs com propósitos distintos. O recibo não inclui nome, texto, prova ou IDs internos. [Contrato HTTP](../spec.md#5-contratos-da-api)
- A revisão reaproveita a identificação e a proteção de saída existentes. A interface distingue admissão, resultado incerto e confirmação, mantém a tentativa em memória e evita um novo UUID durante a recuperação. [Contrato de frontend](../dev-front/f08-story-001.md)

A primeira integração encontrou um erro de binding dos instantes civis. A correção aplica `NpgsqlDbType.Timestamp` aos parâmetros civis e preserva os instantes UTC das operações e da Outbox. A revisão também simplificou a leitura de ativação e removeu um campo redundante da entidade da Outbox.
