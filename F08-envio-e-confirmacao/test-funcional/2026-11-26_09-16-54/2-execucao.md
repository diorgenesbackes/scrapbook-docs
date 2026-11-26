# Aceite funcional local da F08-STORY-005

Em 26/11/2026, o candidato integrado em `dev` passou pelos cenários executados de API, prazo, integridade, recibo, saídas para as funcionalidades dependentes e limpeza. A validação usou PostgreSQL, FFprobe e FFmpeg da imagem do serviço, emulação de S3 e SQS com versionamento, relógio controlado, aviso sintético e eventos descartáveis. O aceite em homologação permanece pendente. [Cartão](https://trello.com/c/KrtTW39U) · [Plano](1-plano.md) · [Spec da F08](../../spec.md#7-estrat%C3%A9gia-de-testes) · [Aceite F08 no PRD](../../../PRD.md#f08-envio-e-confirma%C3%A7%C3%A3o-2)

## Candidato

| Componente | SHA em `dev` |
| --- | --- |
| Serviço | `cb102f4a55a8aeaf7a5e73a77561407e6193152a` |
| UI | `e3b71a564b634f0bd49cab842def52224059a310` |
| Testes | `51f4ca153c9f58469604546123c8c7aecab3f6a1` |
| Infraestrutura | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |
| Base documental | `454142ff367e094d4e122956f43b091cb8b49bf4` |

## Matriz de resultados

| Critério e requisito do PRD | Esperado | Observado e evidência | Estado |
| --- | --- | --- | --- |
| CA1 · F06, F07 e F08 | Depoimento com um áudio válido, duração positiva até 300 s, tamanho de até 50.000.000 bytes e legenda opcional de até 500 escalares. | Confirmados MP3, M4A, WAV, Ogg e WebM, WAV com uma amostra PCM de 0,000125 s, WAV de 300 s, WAV de 50.000.000 bytes e legenda de 500 emojis. Manifestos inválidos de categoria momento, dois arquivos, ausência de bytes, 50.000.001 bytes, 501 escalares, MIME de vídeo e lista de partes foram recusados. A validação rejeitou WAV sem amostra, WAV de 301 s, M4A com faixas defasadas que excedem 300 s, bytes corrompidos, WebM de vídeo declarado como áudio e WAV declarado MP3. A admissão recusou concordância ausente, de outro nome ou de outro evento. | Aprovado no laboratório HTTP. |
| CA1 e CA2 · FR-014, FR-015, F08 e S26 | Usar seleção ou microfone e acompanhar transferência, validação e retomada na mesma tentativa. | Sete jornadas Chromium a 320 px cobriram cinco formatos selecionados, microfone sintético e interrupção do primeiro PUT. O teste manteve a credencial, a chave da tentativa e o objeto de transferência. O progresso usou a quantidade de bytes e a validação teve estado próprio. | Aprovado em dev. [Jornadas E2E](../../test-e2e/f08-story-005-dev.md) |
| CA2 · F05 → F08 | Abertura inclusiva, recusa às 12h, conclusão para tentativa admitida e prova da versão exata até 13h. | A API recusou 1 ms antes da abertura e às 12h. Admitiu no instante exato de abertura. Com prova isolada de 13h + 1 ms, a tentativa terminou rejeitada. Com duas versões, a notificação tardia chegou primeiro e deixou a tentativa em reconciliação. A prova da outra versão às 13h permitiu confirmação posterior. A API manteve o mesmo recibo na repetição. | Aprovado com relógio e notificações de teste. Prova nativa pendente em homologação. |
| CA3 · F08, F09, F10 e F11 | Confirmar após salvar o original privado, manter recibo mínimo e publicar saídas apenas da contribuição confirmada. | Antes da prova, o estado ficou pendente e a Outbox ficou sem fato de mídia. Após a confirmação, registro, sessão e operação estavam confirmados e o recibo tinha somente estado e instante. A versão privada copiada preservou o SHA-256 e o identificador de origem do PUT tempestivo. F09 recebeu um fato com IDs do evento e do registro e instante do recibo. F10 recebeu também o ID do arquivo, que resolveu o original. A consulta pelo contrato de registros confirmou autoria, legenda, categoria, formato, ordem e recebimento do confirmado e excluiu pendente e rejeitado. No pré-voo, a S27 revogou a prévia e retirou as ações de novo envio após o fechamento. | Aprovado para a persistência e os contratos locais. O teste de F11 conferiu os dados no PostgreSQL. |
| CA3 · F08 | Preservar temporários até a elegibilidade e manter intacto o original confirmado. | A versão rejeitada e as duas versões temporárias da tentativa confirmada permaneceram disponíveis durante a observação a 24 h menos 1 ms. Depois de fixar o relógio na elegibilidade, a ausência dos temporários foi observada em 1,0 s para a rejeitada e em 0,3 s para a confirmada. A versão original permaneceu disponível após a limpeza. | Aprovado no relógio do laboratório. |
| CA4 · F08 → F13 | Registrar uma falha por tentativa lógica com etapa, classificação, instante e correlação. | Para o WAV incompatível, a repetição da admissão e da prova preservou uma sessão terminal, a chave lógica, o código `audio-invalid`, o mesmo instante na sessão e na operação e a correlação da admissão. O código identifica a etapa de validação. Para chegada posterior às 13h, o runner verificou o código distinto `audio-received-late`, com terminal e correlação persistidos. | Aprovado para o sinal durável local. O consumo e o alerta de três tentativas pertencem à F13. |
| CA4 · PRD §§2 e 9 | Executar 200 envios simultâneos em dois eventos, com distribuição documentada e p95 de API de até 500 ms sem transferência. | O reteste confirmou 200 envios, 100 por evento, com maior p95 de 434,345 ms nas operações de API medidas. | Aprovado no perfil integrado local. [Ensaio de carga](../../test-performance/f08-story-005-dev.md) |
| CA4 · PRD §§6 e 9 e Spec §7 | Usar aviso aplicável e armazenamento real em homologação. Validar a matriz de dispositivos e codecs. | Foi usado aviso sintético e a emulação recebeu notificações produzidas pelo runner com as versões do bucket local. A matriz física e a integração publicada permanecem para o aceite em homologação. | Pendente em homologação. |

## Execução e evidência

A execução usou `DOCKER_CONTEXT=colima-scrapbook-f08-validation`, `TMPDIR=/private/tmp` e `PYTHONPATH` com o checkout fixo de `scrapbook-tests`. O comando `python3 -B acceptance.py` executou as asserções de `audio_submission_stack.py` e o complemento local. As asserções originais e os primeiros cenários complementares passaram em uma execução de 54,757 s. A execução terminou com código 1 porque a sequência do complemento avançou além da elegibilidade antes de medir o intervalo anterior à limpeza. O reteste reposicionou essa medição e executou `python3 -B acceptance.py --complement-only` sobre os mesmos SHAs. Terminou em 45,410 s, com código 0 e um teste composto aprovado, sem skips.

No preparo, houve uma tentativa sem permissão de bind em loopback, uma com o contexto Docker herdado e duas com o diretório temporário padrão, que impediram a inicialização do serviço na fixture. Essas ocorrências antecederam a execução de cenários de produto. O reteste manteve a VM da F08, usou o diretório montado e removeu seus recursos descartáveis ao encerrar.

| Artefato privado | SHA-256 |
| --- | --- |
| `functional-dev-run-04.log` | `d77ea2de458463a70695de9d5ec103ee744b2e1efc6797117e23c9105d40d9ab` |
| `functional-dev-retest.log` | `0ed530523505de568e254d680252eea1625f3c17fae12e00da27e7f10de15acd` |
| `acceptance.py` | `273363526a9b6b9ae47d232b2fb2702a0d04b7d7a0e67e24325355b829275e6d` |
| `e2e-preflight.log` | `cb844e381f76b53859be031e8f3ce610e8f6248dcb773eb0ec1678691c7436aa` |

Em homologação, a pessoa responsável deve usar o aviso vigente e verificar o S3 versionado, a origem e a precisão da prova nativa, a autorização temporária, notificações tardias e fora de ordem, a recuperação após falha entre cópia e commit e o tempo de limpeza. A matriz física deve cobrir seleção e gravação nos navegadores aplicáveis, inclusive a recuperação quando o microfone está indisponível. [Plano da F08](../../plan.md#fase-5--validar-o-candidato-e-preparar-homologa%C3%A7%C3%A3o) · [Aceite global no PRD](../../../PRD.md#9-crit%C3%A9rios-de-aceita%C3%A7%C3%A3o)
