# Validação funcional da F10-STORY-005

28/11/2026, laboratório descartável. Serviço `d7ac67d`, infraestrutura `0ec49e6`, UI `57f4e63` e Liquibase `ac0e851`. Regressão de navegador em testes `c1782eb`, reteste de pausa em `ab30cec` e reteste de carga em `1157ee1`. [Plano](1-plano.md) · [Especificação](../../spec.md)

| Critério | Resultado local | Aceite em homologação |
| --- | --- | --- |
| CA1 · Prontidão em 300 s | A consulta independente observou 163 de 170 arquivos com commit de disponibilidade em até 72,751 s, e os 170 em até 279,366 s. O relógio monotônico foi ancorado antes da subida da stack, formando um limite superior para a prontidão após confirmação. [Medição](../../test-performance/f10-story-005-dev.md) | Pendente o ensaio com AWS e medição individual entre `confirmedAt` e commit. |
| CA2 · Carga e recibos | Após corrigir o relógio das notificações sintéticas, o reteste registrou 200/200 recibos F08 e 170 arquivos, sendo 90 fotos, 40 áudios e 40 vídeos. As quatro operações comuns ficaram com p95 abaixo de 500 ms no cenário local. [Medição](../../test-performance/f10-story-005-dev.md) | Pendente a rodada de borda, a observação do recibo no ambiente publicado e a capacidade com quotas reais. |
| CA3 · Saídas e telemetria | Quatro jornadas Chromium passaram. JPEG, PNG e HEIC de entrada, conversão de foto, privacidade das variantes e sinais com dimensões fixas foram verificados. O perfil misto exercitou a fixture de contrato para áudio/vídeo. [Jornadas](../../test-e2e/f10-story-005-dev.md) · [Privacidade](../../test-security/f10-story-005-dev.md) | Pendente a inspeção de S3, MediaConvert, EventBridge, SQS, IAM/KMS e CloudWatch reais. |
| CA4 · Ativação e recuperação | Dois ciclos de pausa/retomada de foto passaram no reteste, com recibos e originais preservados, quatro variantes prontas, dois fatos publicados e fila drenada. Reconciliação de tentativas de áudio/vídeo em pausa verificada nos unitários. [Jornadas](../../test-e2e/f10-story-005-dev.md) · [Revisão](../../code-review/f10-story-005.md) | Pendentes a ativação gradual, a drenagem com concorrência real, o ensaio de histórico no ambiente e o rollback de versão. |

O comportamento integrado local segue os critérios aplicáveis ao laboratório. O aceite integral depende do candidato publicado e do ensaio com acesso autorizado à AWS. [Entrega e dependências](../../delivery/f10-story-005.md)
