# F03-STORY-003 · Desenvolvimento de backend

O comando `PUT /api/v1/events/{eventId}/details` normaliza nome, data e horário de Brasília e boas-vindas. O serviço consulta o vínculo com bloqueio da linha, revalida a versão e a trava estrutural e grava detalhes e três marcos em uma transação. Uma abertura programada incompatível devolve erro em `startsAt`. A edição mantém o código público e a versão da política.

A trava temporal é avaliada a partir do corte anterior. Se a janela anterior ainda estava flexível e a nova já terminou, o serviço materializa a transição com o novo corte. Um marco persistido é preservado.

A criação incorpora a mensagem personalizada ao hash idempotente canônico. A opção padrão mantém o formato do hash anterior, inclusive quando o cliente envia `null` explicitamente. `usesDefaultWelcomeMessage` permite ao cliente distinguir a opção padrão de um texto personalizado idêntico. As colunas já existem na base do Liquibase.

[Contrato da F03](../spec.md#5-contratos-de-api) · [Revisão técnica](../code-review/f03-story-003.md)
