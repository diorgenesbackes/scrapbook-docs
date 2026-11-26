# Verificação defensiva do envio de vídeo em dev

A revisão cobre a autorização da tentativa e dos arquivos, o armazenamento temporário, a prova de prazo do S3, o FFprobe e a confirmação com o original privado. O laboratório local usa PostgreSQL e Moto descartáveis, relógio sintético e origens em loopback. As verificações dinâmicas usam dois eventos próprios e volume baixo. A consulta externa é guiada pelo contrato e pelo conhecimento prévio dos controles. [Especificação da F08](../spec.md) · [Segurança e dependências](../../guidelines/principios.md#5-contratos-dados-e-segurança)

## Achado SEC-F084-01

O Dockerfile do serviço em `7f11a172ecb0056e712dca3435319a8871df6f01` instalava o FFmpeg Noble `7:6.1.1-3ubuntu5` ao montar a imagem. A Canonical identifica essa linha como afetada pela CVE-2025-22921 no decoder JPEG 2000 e aponta a correção para `7:6.1.1-3ubuntu5+esm3`. O processo de inspeção aceita MOV e abre a faixa de vídeo em um decoder nativo para conferir um quadro. A falha pode interromper esse subprocesso. Gravidade contextual média e alta confiança na versão afetada e no caminho de inspeção. A constatação usa revisão da fonte e confirmação do fornecedor. [Canonical](https://ubuntu.com/security/CVE-2025-22921) · [Correções do FFmpeg](https://www.ffmpeg.org/security.html#ffmpeg-8.0)

O ciclo inicial substituiu o pacote pelo FFmpeg oficial 9.1, compilado na arquitetura alvo com arquivo, assinatura e chave pública fixados por SHA-256. A construção verifica a assinatura com GnuPG. A compilação limita os demuxers ao MOV e ao Matroska usado para WebM, restringe o protocolo a arquivo e desabilita a rede. O build inclui dav1d 1.5.4 para AV1 e conserva os decoders nativos. O arquivo oficial do FFmpeg passou na conferência OpenPGP RSA/SHA-512 com a impressão digital publicada `FCF986EA15E6E293A5644F10B4322F04D67658D8`. O SHA-256 do dav1d coincidiu com o publicado pelo VideoLAN. O código-fonte assinado contém a correção publicada para a CVE-2025-22921. O build Linux ARM64 em `95d1a90` confirmou a assinatura no GnuPG e as opções compiladas. O fluxo integrado em ARM64 passou no reteste descrito abaixo. A execução AWS e o build Linux AMD64 ficam para a validação do candidato publicado. [Release e verificação do FFmpeg](https://ffmpeg.org/download.html) · [Distribuição do dav1d](https://download.videolan.org/pub/videolan/dav1d/1.5.4/)

O candidato anterior tratava os códigos 134, 135, 136 e 139 do subprocesso como indisponibilidade do decoder. A tentativa entrava em nova execução a cada 30 s e poderia ficar em validação indefinidamente. A correção `cbc1225b6a96993a02646dd85c75bf0fc40ef3d7` classifica esses estados como conteúdo inválido, que segue para a rejeição e a limpeza. O reteste em .NET 10.0.401 usou um processo sintético que devolve códigos fixos. Os dez casos passaram. Os códigos 125–127 continuam recuperáveis como falha operacional e 137 e 152 continuam classificados como esgotamento de orçamento.

## Matriz de cobertura

| Perspectiva | Controle e evidência | Estado |
| --- | --- | --- |
| White box | `SubmissionService` consulta pelo evento e pela chave da tentativa, autentica HMAC contextual e expiração, e passa ID autorizado ao repositório. `PhotoSubmissionRepository` exige o registro e a chave do arquivo. | Aderente na fonte. |
| White box | `VideoSubmissionService` confere manifesto, sequência, tamanho exato, ETag e SHA-256 da parte. A assinatura do PUT recebe a operação, o host, a chave, o ID multipart, a parte e os cabeçalhos de tamanho e checksum. O prazo fica limitado a dez minutos, à validade da credencial e ao limite da tentativa. A UI valida a origem configurada e aplica CSP. | Aderente na fonte. Aplicação pelo AWS pendente. |
| White box | O consumidor aceita operações previstas e associa bucket, chave, versão e instante da prova. O worker lê a versão fixada em fluxo limitado, confere tamanho, hash integral e mídia real, copia a versão validada para chave privada e confere o original antes do commit com a Outbox. O sinal de conclusão apenas pede reconciliação. | Aderente na fonte. |
| White box | O processo do FFprobe recebe arquivo temporário privado, argumentos fixos e ambiente esvaziado. Há limites de tempo, CPU, memória, pixels, streams, saída e paralelismo. O Compose usa usuário da aplicação, raiz somente leitura e remove capabilities. O processo compartilha usuário e contêiner com a API. | Limites presentes. Correções do SEC-F084-01 verificadas na fonte, no build ARM64 e no reteste local. |
| White box | CloudFormation define versionamento, bloqueio de acesso público, SSE-KMS, propriedade do bucket e CORS de PUT. Escrita em originais depende de cópia de temporários. O executor só lê versões originais. A fila restringe origem da notificação ao bucket e à conta. | Configuração inspecionada. Permissões efetivas AWS pendentes. |
| White box | API e proxy usam modelo de rota nos logs. Mensagens de worker e Problem Details usam códigos e correlação. Conteúdo, hash, segredo e URL assinada ficam fora dos campos de log inspecionados. | Aderente na fonte. |
| Grey box | Duas tentativas válidas devolveram estado e status de upload mínimos. Onze respostas para tentativa, evento ou arquivo alheio, segredo divergente e credencial ausente retornaram 404 equivalentes a um recurso ausente. Quatro operações com credencial expirada retornaram 404. | Aprovado no laboratório local. |
| Grey box | A emissão rejeitou parte fora do manifesto, propriedade duplicada e corpo acima de 1 KiB. A parte final autorizada foi reconhecida com a assinatura esperada. Nova autorização no limite foi recusada. A conclusão sem transferência devolveu 202, sem objeto, recibo ou Outbox. | Aprovado no laboratório local. |
| Grey box | Um MP4 válido gerou recibo com estado e instante e uma versão privada com o SHA-256 integral. O mesmo arquivo declarado WebM foi rejeitado sem versão no original. Quinze marcadores de dados privados ficaram fora dos logs da API e do proxy. | Aprovado no laboratório local. |
| Black box guiada | Respostas anônimas da API omitiram metadados e mantiveram `no-store` e correlação. A UI enviou CSP com a origem de upload configurada, `frame-ancestors 'none'`, `nosniff` e `no-referrer`. O GET anônimo ao original no Moto retornou 403. | Aprovado no laboratório local. |

## Revisões e execução

| Repositório | SHA em dev |
| --- | --- |
| Serviço | `203cc8365c326bf542aa08aa211f3dd5c7b3a9af` |
| UI | `b34aa71b21149f5481b199813f9b5e767c1dc3c4` |
| Infraestrutura | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |
| Testes | `1593873a095d7a1a9583775608eb8d587c63f406` |

O reteste de 26/11/2026 usou Python 3.13.15 e Docker Compose 5.6.0, com a fixture PostgreSQL 18.6 e Moto do repositório de testes em loopback. O caso dirigido usou a mesma preparação da suíte de vídeo e passou em 29,346 s com código de saída 0. As primeiras tentativas ficaram bloqueadas por espaço no disco da VM descartável. Após a expansão, uma asserção do ensaio foi ajustada para avaliar as diretivas do cache, pois a resposta válida continha `no-cache,no-store`. O reteste passou na mesma revisão do serviço. [Preparação do ambiente local](../spec.md#7-estratégia-de-testes)

## Homologação

- Na revisão de configuração efetiva, conferir a versão implantada do serviço e do FFprobe, versionamento, KMS, IAM de runtime, políticas da fila e do bucket, CORS, logs da borda e orçamento de armazenamento temporário. A inspeção de CloudFormation cobre a configuração declarada.
- Com identidades e eventos sintéticos próprios, conferir o PUT e as partes no S3 real com tamanho, SHA-256 e validade, a prova nativa de conclusão, a cópia da versão exata, a ausência de gravação direta no prefixo de originais e a limpeza das versões e multiparts elegíveis.
- A partir de navegador e HTTP, verificar CSP, CORS, ausência de acesso público ao original, resposta sem credencial e corpo, cache e cabeçalhos dos erros. Registrar também a lista de codecs comprovados nos dispositivos previstos. [Estratégia de testes](../spec.md#7-estratégia-de-testes)
