# Aceite funcional local da F08-STORY-004

Em 26/11/2026, o candidato integrado em `dev` passou nos cenários locais executados de envio, retomada, prazo, recuperação e limpeza. O aceite funcional integral segue pendente da transferência de um vídeo no limite de tamanho, da validação de dispositivos e da integração com o S3 real. A evidência local sustenta entregar o candidato para esses ensaios em homologação. [Cartão](https://trello.com/c/UR7dLziQ) · [Plano](1-plano.md) · [Spec da F08](../../spec.md#1-visão-técnica) · [Aceite da F08 no PRD](../../../PRD.md#f08-envio-e-confirmação-2) · [Fluxo do HLD](../../../HLD.md#fluxo-de-requisições-e-de-dados)

## Versões e execução

| Componente | SHA conferido no checkout |
| --- | --- |
| Serviço | `203cc8365c326bf542aa08aa211f3dd5c7b3a9af` |
| UI | `b34aa71b21149f5481b199813f9b5e767c1dc3c4` |
| Infraestrutura | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |
| Testes | `1593873a095d7a1a9583775608eb8d587c63f406` |
| Base documental | `2088ee6c108b79b6af3def31c0f64ee78b4d077e` |

A validação funcional conferiu as asserções do runner e os artefatos da execução `python3 -B video_submission_stack.py --with-browser` em `scrapbook-tests`. A execução encerrou com código 0 em 101,806 s. O cenário HTTP reúne as verificações de API, armazenamento e PostgreSQL. O JUnit registrou seis jornadas em 40,796 s, sem falhas, erros ou skips. As primeiras tentativas integradas pararam antes da validação funcional no preparo do PostgreSQL. Um dos logs registrou falta de espaço na VM descartável. O reteste ocorreu após ampliar o disco do perfil de 24 para 48 GiB. [Evidência E2E](../../test-e2e/f08-story-004-dev.md)

O laboratório usou PostgreSQL 18.6, o worker e o FFprobe da imagem do serviço, Chromium desktop a 320 × 740, dois eventos e mídia sintética. Moto forneceu bucket versionado e fila em loopback. A fixture calcula o SHA-256 das partes que o Moto omite em `ListParts` e publica notificações com a versão efetivamente encontrada no armazenamento. O relógio controlado reproduz os marcos em Brasília. [Fixture](../../dev-infra/f08-story-004.md)

## Resultados por critério

| Critério e referência | Resultado observado no candidato | Estado |
| --- | --- | --- |
| **CA1 · Categoria, formato, duração e legenda.** F07 e F08 do PRD. | As quatro jornadas com seleção confirmaram MP4 de 60 s em momento, MP4 de 300 s em depoimento, MOV em depoimento e WebM em momento. O formulário habilitou o envio após a concordância e enviou um vídeo, com categoria, tamanho, checksum e legenda de 500 escalares. A gravação com MediaRecorder e câmera sintética abriu revisão e confirmou sem legenda. Por HTTP, WebM de 60 s e MOV de 300 s foram confirmados. Um MOV de 300 s como momento e um WebM de 301 s como depoimento foram rejeitados. O MOV válido preservou a rotação normalizada no banco. A verificação defensiva também confirmou a recusa de MP4 declarado como WebM. [E2E](../../test-e2e/f08-story-004-dev.md) · [Segurança defensiva](../../test-security/f08-story-004-dev.md) | Aprovado nesses cenários locais. |
| **CA1 · Limite de tamanho e fronteiras de admissão.** F07 e F08 do PRD. | Um manifesto de 500.000.001 bytes recebeu HTTP 400 com `REQUEST_VALIDATION_FAILED`. Os unitários cobrem admissão de 500.000.000 bytes, zero bytes, quantidade diferente de um e legenda de 501 escalares. O multipart funcional usou um MP4 de 33.554.533 bytes. | Aprovada a recusa HTTP acima do limite. Transferência e validação do arquivo inteiro de 500.000.000 bytes não executadas. |
| **CA2 · Retomada da mesma tentativa e progresso na S26.** FR-014 e FR-015, F08 do PRD. | Depois do primeiro PUT, o runner moveu o relógio 11 minutos e a API reconheceu somente a parte 1. A API recusou renovar essa parte com HTTP 409 e emitiu autorizações válidas para as demais. O navegador simulou perda da conexão no primeiro PUT da parte 2. Ao retomar, recuperou a parte 1 pelo status, iniciou a barra em 16.777.216 bytes e enviou somente as partes faltantes. Houve um POST de admissão e a credencial e a tentativa se mantiveram nas consultas, autorizações e conclusão. | Aprovado no laboratório. A expiração efetiva de uma URL pelo S3 depende do provedor publicado. |
| **CA3 · Lista de partes, versão e prazo inclusivo.** F05 → F08 no PRD. | Com só a parte 1, a listagem trouxe essa parte, sem versão concluída ou recibo. A conclusão com todas as partes criou uma versão temporária. Repetir a chamada retornou HTTP 202 e preservou a mesma versão. A tela mostrou validação e depois verificação de recebimento antes da prova. O evento dessa versão às 13h foi recebido depois do prazo e permitiu confirmar. O evento às 13h + 1 ms foi registrado e a tentativa acabou em `rejected` após a reconciliação. | Aprovado com relógio e notificação do laboratório. Prova nativa pendente em homologação. |
| **CA4 · Integridade, original e recibo após commit.** FR-015, F08 → F09/F10 no PRD. | Um bloqueio de commit observável no PostgreSQL manteve a tentativa em `validating` após a cópia privada. Ao retirar a restrição, o worker preservou a mesma versão copiada. A consulta ao banco conferiu a versão temporária fixada, a operação `complete_multipart`, o SHA-256 integral esperado, o vínculo com a versão original e um fato de mídia para o arquivo. Repetir a admissão devolveu HTTP 200 e o mesmo recibo. O navegador consultou o status com `no-store` e exibiu horário de Brasília sem conteúdo, autoria ou ações de mídia no recibo. A prévia local foi revogada e, após o prazo, a interface retirou as ações de novo envio. | Aprovados a recuperação do commit, a repetição da conclusão e o recibo no laboratório. Falha de comunicação com resultado incerto durante a chamada ao provedor pendente em homologação. |
| **CA4 · Limpeza de partes e versões.** F08 do PRD. | Com o relógio avançado dois dias e um minuto além das 13h, a tentativa com somente a primeira parte expirou. A listagem do storage confirmou o cancelamento do multipart e a ausência de versões temporária ou original dessa tentativa. A limpeza da tentativa confirmada removeu suas versões temporárias e preservou a mesma versão original privada. | Aprovada a limpeza de elegíveis no laboratório. O ensaio avançou além de 24 h e a medição do atraso operacional na borda continua pendente. |

A unidade de negócio é byte decimal. As referências de 32 MiB e 16 MiB são, respectivamente, o limiar técnico de multipart e o tamanho da parte. Na evidência local, a verificação de posse da tentativa também devolveu HTTP 404 para segredo alheio e pedido de autorização em outro evento. Os efeitos consumidores da F09 e da F10, a galeria da F11 e o alerta da F13 pertencem às entregas que os integram. [Contratos da F08](../../spec.md#7-estratégia-de-testes) · [Integrações no PRD](../../../PRD.md#integração-entre-funcionalidades)

## Aceite em homologação

- Conferir o S3 versionado, a origem das notificações, IAM, SSE-KMS, CORS e a validade efetiva das autorizações SigV4. Exercitar a prova nativa de conclusão no limite, sua entrega tardia e duplicada, a associação à versão exata, a perda de resposta na conclusão ou na cópia e o atraso de limpeza a partir da elegibilidade. [Infraestrutura](../../dev-infra/f08-story-004.md) · [Candidato](../../delivery/f08-story-004.md)
- Confirmar um vídeo válido de 500.000.000 bytes de ponta a ponta e verificar espaço temporário e orçamento do FFprobe. Em Android e iPhone físicos, validar captura e seleção com os containers e codecs aplicáveis, incluindo o fluxo com permissão de câmera negada. A captura sintética no Chromium comprova o fluxo de revisão e upload nesse navegador. [Restrições e testes da F08](../../spec.md#7-estratégia-de-testes) · [Compatibilidade no PRD](../../../PRD.md#f01-base-da-aplicação-e-ambientes-2)

## Artefatos da execução

| Evidência privada identificada pelo nome | SHA-256 |
| --- | --- |
| `video-suite-04.log` | `bb50968f09deb8d39de437281992823fecdd33fc5bf98868dd96205c04f36b21` |
| `test-results/video-submissions.xml` | `fd8ee11f863e598f882d628e86a5e1452ef082d419d5e7a6352df25cd28bd486` |

Os nomes das asserções HTTP estão em `video_submission_stack.py` e as jornadas de navegador em `video-submissions/video-submission.spec.ts`, no SHA de testes acima. Este relatório usa os resultados existentes do candidato integrado e registra as pendências de cobertura.
