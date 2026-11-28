# F10-STORY-004 — Recuperação operacional de mídias

O runner usa o fato canônico da Outbox ou o comprovante persistido da F08, incluindo a identidade determinística por arquivo, o instante de confirmação e o original versionado com checksum. Registra casos inconclusivos sem modificar o trabalho existente. A varredura pagina por ID e usa um instante fixo de ativação em UTC. Cada réplica reserva uma das duas vagas de foto e uma das duas vagas de áudio/vídeo para confirmações normais. Na vaga flexível, duas rodadas dão preferência a confirmações novas e uma dá preferência ao histórico. A Outbox publica primeiro os fatos confirmados a partir da ativação, e o consumidor aplica o mesmo corte ao classificar os trabalhos. [Especificação](../spec.md) · [Cartão](https://trello.com/c/dLVALXU9)

O redrive mantém o `messageId`, o estado persistido e o ciclo da falha. Até 30 dias inclusive, um fato canônico pode admitir trabalho ausente. Após a janela, a operação só restaura a deduplicação se o efeito persistido tiver a identidade comprovada. Se a Outbox tiver sido arquivada, o runner cruza o trabalho, o recibo e o original antes de reconstruir a proteção. Retenha a deduplicação por pelo menos 37 dias desde a confirmação e enquanto existir estado recuperável. Preserve essa proteção também durante a reentrega nativa de mensagens. [Retenção e reprocessamento](../../guidelines/mensageria.md)

A reabertura exige falha definitiva, perfil ativo, ciclo esperado e confirmação comprovada. Qualquer tentativa ativa ou ambígua impede a reabertura. O novo ciclo reutiliza o trabalho, aumenta `retry_cycle`, reinicia o contador do ciclo e continua a sequência de números de tentativa. O fato definitivo da F13 usa a identidade do ciclo. A projeção para a F11 exige o evento, registro e arquivo e retorna referências versionadas do perfil ativo apenas com o conjunto pronto completo.

## Comando operacional

Aplique a geração `f10-story-004-media-recovery`. Provisione um login PostgreSQL exclusivo, com `CONNECT` no banco e associação à role `scrapbook_media_operator` com `INHERIT TRUE, SET FALSE`. Passe a conexão privada em `ConnectionStrings__ScrapBookMediaRecovery`. Habilite o processo explicitamente com `MediaRecovery__Enabled=true`. Defina `MediaRecovery__HistoryBeforeUtc` com o mesmo instante fixo da ativação no serviço e no runner, em ISO 8601 com `Z` ou `+00:00`. Para limpeza, passe também `PhotoSubmissions__Bucket`, `PhotoSubmissions__Region` e credenciais da role de limpeza do ambiente.

```text
dotnet ScrapBook.Api.dll media-recover history EVENTO CURSOR LIMITE INCIDENTE REVISAO
dotnet ScrapBook.Api.dll media-recover redrive EVENTO MESSAGE_ID INCIDENTE REVISAO
dotnet ScrapBook.Api.dll media-recover reopen EVENTO ARQUIVO PERFIL CICLO_ESPERADO INCIDENTE REVISAO
dotnet ScrapBook.Api.dll media-recover cleanup EVENTO CURSOR LIMITE INCIDENTE REVISAO
```

Use o código do incidente e o SHA da correção. A ausência de `--apply` simula a operação. A simulação de redrive e reabertura pode registrar o diagnóstico na auditoria, sem avançar o trabalho. Em lotes, comece com cursor `0`, limite entre 1 e 100 e use o `nextId` retornado. Revise as contagens antes de aplicar. Quando um item ficar deferido, revisite o trecho a partir do cursor anterior em outra execução. Aplique com `--apply` ao final do comando. A auditoria registra ação, resultado, operador, causa e revisão.

A limpeza aguarda 24 horas após o encerramento da tentativa. Se houver uma versão registrada no objeto, usa apenas essa versão. Para um artefato planejado, a listagem deve comprovar uma única versão antiga da chave exata. A intenção `cleanup_jobs` registra a versão antes de qualquer exclusão. O lease permite retomar com a mesma versão após interrupção. Referências prontas de qualquer perfil, originais e recibos ficam fora da seleção. A revisão de casos com versões não registradas e ambíguas fica com a operação. [Modelo de dados](../spec.md#6-modelo-de-dados)

## Reconciliação do provedor

O worker mantém a mesma identidade e consulta o resultado pelo identificador externo conhecido ou pela janela e fila da submissão. Após 24 horas inconclusivas, o erro `MEDIA_RECONCILIATION_INTERVENTION_REQUIRED` permanece no trabalho para triagem. Consulte os jobs de `av_transcode` em `reconciling` com esse erro. Restrinja o diagnóstico ao evento e compare a tentativa, sua janela e a fila esperada com o provedor. Preserve a intenção enquanto o resultado seguir incerto. [Contrato de áudio e do worker compartilhado](f10-story-002.md)
