# Execução funcional local da F10-STORY-004

A validação local aprovou os cenários de CA1 a CA4. Em CA2, a resposta HTTP 503 manteve a tentativa recuperável e sem ID de provedor. Depois de vencer a janela de submissão, o serviço consultou e recuperou o mesmo job sem receber callback ou enviar outro `CreateJob`. Na simulação anterior de resposta perdida por fechamento da conexão, a fixture recebeu duas chamadas com a mesma chave e criou um único job. A origem da segunda chamada e a indisponibilidade inicial do redrive seguem sem causa isolada.

A execução avaliou o serviço integrado em `dev` em 28/11/2026. Os dados são sintéticos e cada runner recebeu um projeto Compose próprio. PostgreSQL 18.6 armazenou os efeitos reais dos comandos e workers. Moto emulou S3 versionado, SNS e SQS. A fixture do MediaConvert trabalhou com o contrato HTTP e artefatos de áudio e vídeo sintéticos. [Cartão](https://trello.com/c/dLVALXU9) · [Spec](../../spec.md#aceite-rastreado-ao-prd) · [Plano](1-plano.md) · [Regressões no navegador](../../test-e2e/f10-story-004-dev.md)

| Repositório | SHA |
| --- | --- |
| `scrapbook-service` | `3834529e86e1123a4c3119c67be5395478ea6612` |
| `scrapbook-infrastructure` | `8f6dd8cfb4226c467c48fc63e34667a82fb4b17a` |
| `scrapbook-tests`, rodada inicial | `a62469c3711a27312c9e184ab72eedc91ff7b4ff` |
| `scrapbook-tests`, reteste da fixture | `4a720663ae2f1a398ba941bb2f7c10a6cc674822` |
| `scrapbook-docs` | `c6eba10e48ece6d0b54b2635539ae3eef5591223` |
| `scrapbook-ui` | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| `scrapbook-liquibase` | `ac0e8517af47cdf4a61b81c715b0e8544040ecde` |

| Critério | Esperado | Observado |
| --- | --- | --- |
| CA1 · F08 → F10 | Recuperar arquivos antigos com prova, preservar os recibos e repartir a capacidade com novos envios. | **Aprovado.** A simulação preservou as tabelas. Páginas de um item admitiram duas confirmações, pela Outbox e pelo comprovante F08, e rejeitaram outra inconclusiva, com auditoria. A foto nova ficou pronta durante a pausa de promoção do item histórico. |
| CA2 · Recuperação | Reconvergir após lease vencido, perda ou reordenação de resultado e redrive. | **Aprovado no laboratório.** A retomada de duas réplicas após perda do lease produziu uma única promoção. Resultados antecipados, duplicados e tardios preservaram tentativa e versão. A reentrega da DLQ preservou o ID e a deduplicação. O redrive passou em dois retestes. Com resposta HTTP 503, o serviço aguardou em `unknown` e recuperou o mesmo job por `ListJobs` e `GetJob` após a janela, com um `CreateJob` e os callbacks ainda retidos. |
| CA3 · F10 → F13 | Simular e reabrir ciclo elegível com auditoria, preservando trabalho e arquivo e emitindo uma falha definitiva por ciclo. | **Aprovado.** A simulação preservou o ciclo inicial e o CLI exigiu o ciclo esperado. Os ciclos 1 e 2 produziram dois fatos definitivos distintos com etapa, instante e correlação da confirmação. A segunda reabertura levou o ciclo 3 à prontidão, preservando trabalho, recibo e original. |
| CA4 · F10 → F11 | Restringir as referências ao perfil ativo e às variantes prontas do arquivo correto e limpar versões específicas elegíveis. | **Aprovado no reteste.** O contrato interno devolveu `pending`, `processing`, `available` e `failed` por arquivo. A consulta devolveu somente o conjunto completo do perfil ativo, com versões exatas no S3. IDs de outro evento, registro ou arquivo retornaram ausência. Na limpeza, excluiu-se somente a versão cadastrada após cumprir a retenção. O objeto ambíguo ficou deferido e as duas versões sem vínculo, o original, o recibo e as variantes prontas foram preservados. |

## Rodadas de recuperação

Comandos executados em `scrapbook-tests` com `DOCKER_CONTEXT=colima-scrapbook-f10-validation`, `TMPDIR=/private/tmp` e fuso de São Paulo.

```sh
python3 -B media_recovery_stack.py history
python3 -B media_recovery_stack.py redrive-reopen
python3 -B media_recovery_stack.py cleanup
```

- `history` retornou `0` em 44,100 s.
- `redrive-reopen` retornou `1` em 30,464 s. O primeiro `redrive --apply` após o preview retornou o código operacional `MEDIA_RECOVERY_UNAVAILABLE` com saída `4`, no lugar de `MEDIA_REDRIVE_ADMITTED` com saída `0`. A mesma sequência passou em 54,406 s no reteste sem sobreposição com outras suítes. A instrumentação foi adicionada à cópia temporária do runner para observar apenas classes e códigos de exceção. Nessa rodada, o redrive e a reabertura passaram em 0,523 s e 0,508 s, respectivamente, sem exceção capturada. O runner padrão também voltou com `0` em 38,501 s no SHA `4a720663`. A causa da falha inicial segue aberta.
- `cleanup` retornou `1` em 40,411 s. Na comparação de preparação, o Moto atribuiu à gravação posterior um horário 20 s anterior a uma gravação já observada. A suíte parou antes do cenário de exclusão. O [PR #23 de testes](https://github.com/diorgenesbackes/scrapbook-tests/pull/23) removeu a dependência de ordenar gravações pelo relógio da VM e preservou as asserções de retenção, exclusão da versão cadastrada e preservação das demais. O reteste retornou `0` em 38,883 s.
- Uma sonda temporária retornou `0` em 59,591 s. Uma fila de origem exclusiva do Moto encaminhou o envelope confirmado à DLQ após uma entrega de teste sem ACK. A sonda conferiu o corpo e o `messageId` e reentregou o mesmo envelope ao consumidor. Para a projeção, a cópia temporária da API executou `MediaPreparationService.GetFileAsync` com o repositório e PostgreSQL reais, sem introduzir rota pública. O ensaio verificou ainda dois IDs de falha F13 em ciclos distintos e a recuperação no terceiro ciclo.

## Sonda de reconciliação sem entrega de callback

As sondas rodaram no integrado com testes em `4a720663`. A fila de resultado do Moto aplicou atraso de 900 s. O relógio controlado da aplicação permitiu observar a reconciliação após a janela de 45 s de submissão.

- **ID do provedor conhecido.** A sonda persistiu a associação antes de concluir o job na fixture. `GetJob` levou o arquivo à prontidão com a mesma tentativa, mesmo job, original e recibo preservados. O reteste confirmou dois callbacks retidos, sem mensagens visíveis ou em trânsito.
- **Resposta de `CreateJob` perdida.** A primeira rodada retornou `1` em 74,604 s após esgotar a espera pelo estado sem ID. No reteste, a primeira observação com uma tentativa já encontrou `available/completed/succeeded/ready`, com ID persistido, ainda antes de avançar o relógio em 2 min para esse arquivo. A API vinculou a saída versionada ao mesmo job registrado na fixture e preservou o original e o recibo. Os quatro callbacks permaneciam retidos, sem mensagens visíveis ou em trânsito. A fixture registrou um job por arquivo, com 1 chamada `CreateJob` no primeiro e 2 com a mesma chave no segundo. A asserção de uma chamada por arquivo falhou, com saída `1` em 31,407 s. A sonda comprova prontidão sem entrega de callback e deduplicação do efeito. O registro dessa rodada não identifica a origem da segunda chamada, sua ordem em relação a `ListJobs` ou a persistência intermediária do estado sem ID.
- **Resposta HTTP 503.** A cópia temporária da fixture aceitou o job, reteve os callbacks e devolveu 503 com corpo. O serviço deixou o arquivo em `processing/reconciling/unknown` com uma tentativa e sem ID de provedor. O rastro sintético UTC registrou `CreateJob` às 17:19:47 e, 46 s depois, `ListJobs` e `GetJob` às 17:20:33. O arquivo ficou pronto com o mesmo job e a mesma tentativa, preservando original e recibo. A fila ainda tinha duas mensagens retidas e zero visíveis ou em trânsito. A fixture contou um único `CreateJob`. O ensaio terminou com saída `0` em 28,610 s.

A entrega real do MediaConvert, dos eventos e das políticas de IAM/S3, o redrive nativo de filas da AWS e a meta do perfil misto em 300 s exigem validação correspondente em homologação. A galeria autorizada e o envio de alertas pertencem às integrações da F11 e da F13.
