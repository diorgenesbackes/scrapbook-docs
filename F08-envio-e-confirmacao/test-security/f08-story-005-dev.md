# Verificação defensiva do envio de áudio em dev

Auditoria da F08-STORY-005 em escopo local. A inspeção acompanhou a credencial de tentativa, o PUT direto e a prova de recepção, a validação em subprocesso, a confirmação com original privado e a limpeza. A observação HTTP é guiada pelo contrato e pela revisão da fonte. As fixtures usam eventos, nomes, conteúdo, segredos e relógio sintéticos. [Especificação F08](../spec.md) · [Contrato de áudio](../dev-back/f08-story-005.md)

| Repositório | SHA integrado em dev |
| --- | --- |
| Serviço | `cb102f4a55a8aeaf7a5e73a77561407e6193152a` |
| UI | `e3b71a564b634f0bd49cab842def52224059a310` |
| Testes | `51f4ca153c9f58469604546123c8c7aecab3f6a1` |
| Docs | `454142ff367e094d4e122956f43b091cb8b49bf4` |
| Infraestrutura | `006e860c2fb5e78b8942226398ab2bc53590ed15` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |

## Inspeção da fonte

| Controle | Evidência | Resultado |
| --- | --- | --- |
| Credencial, escopo e revogação | `SubmissionController` exige a credencial nas operações da tentativa. `SubmissionService` consulta por evento, chave e operação, verifica HMAC contextual em tempo fixo e a validade da credencial. `PhotoSubmissionRepository` consulta pelo registro autorizado e chave do arquivo. `PhotoSubmissionService` impede novas autorizações após a prova fixada, saída do estado de recepção ou prazo de transferência. | Aderente na fonte. |
| PUT privado e prova de recepção | `S3PhotoSubmissionStorage` assina PUT para host e chave temporária opaca, com tipo, comprimento e SHA-256. O serviço limita validade a dez minutos, ao prazo de conclusão e à validade da tentativa. A prova correlaciona operação, bucket, chave, versão e instante. O sinal de conclusão retorna estado e a confirmação depende da prova. | Aderente na fonte. |
| Bytes, mídia e cópia privada | `S3VideoObjectReader` exige versão e tamanho exatos e calcula SHA-256 em fluxo limitado. `AudioSubmissionService` compara o hash, verifica tipo de mídia, duração e codec e verifica SHA-256 do original. `PhotoSubmissionService` copia a versão validada, verifica a cópia e confirma no mesmo commit do fato de mídia. | Aderente na fonte. |
| Execução do decodificador | `AudioDecoderProcess` usa argumentos fixos, ambiente reduzido, diretório temporário privado, protocolo de arquivo e limites de tempo, streams, alocação e saída. Em Linux, `prlimit` limita CPU e espaço de endereçamento. O build fixa o arquivo e a chave do FFmpeg 9.1 e verifica a assinatura. O código do FFmpeg com o SHA-256 pinado confirma que referências e caminhos absolutos do MOV começam desabilitados. | Limites verificados na fonte. |
| Privacidade de saída e logs | `SubmissionStatusResponse` devolve somente estado e instante quando confirmado. Logs da API e Nginx usam rotas nominais, correlação e códigos técnicos. No navegador, `submissionUpload` limita a origem à configurada e confere método e cabeçalhos esperados. A UI mantém a tentativa em memória e limpa a referência no estado terminal. | Aderente na fonte. |
| Limpeza e configuração declarada | A seleção usa o estado terminal, a janela de retenção e a versão temporária correlacionada. A infraestrutura declara versionamento, bloqueio de acesso público, SSE-KMS, CORS de PUT, origem autorizada da notificação e separação de permissões de temporários e originais. | Aderente na fonte. Aplicação AWS em homologação. |

## Verificação no laboratório

Em 26/11/2026, um ensaio serial em loopback passou em 31,575 s, com código de saída 0. O ensaio em Python 3.13.15 reaproveitou `AudioSubmissionStackTests` de `audio_submission_stack.py` em uma stack descartável com Docker 29.8.2, Compose 5.6.0, PostgreSQL 18.6 e Moto. O relógio sintético e as notificações de PUT da fixture controlaram a chegada, o prazo de emissão, a expiração e a elegibilidade da limpeza. O runner temporário executou `python3 check_controls.py <checkout-de-testes-em-dev>`, com `DOCKER_CONTEXT=colima-scrapbook-f08-validation` e `TMPDIR=/private/tmp`.

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| Grey box | As duas tentativas legítimas devolveram os estados, autorizações e sinais de conclusão previstos. Treze respostas para credencial ausente ou de outra tentativa, evento divergente, referência de arquivo divergente e replay não autorizado tiveram o mesmo corpo seguro e código 404 de um recurso ausente, exceto pelos identificadores de correlação e tracing. | Aprovado no laboratório. |
| Grey box | As duas autorizações retornaram PUT para uma chave temporária no host configurado. A assinatura cobriu host, comprimento, MIME e SHA-256. A validade ficou dentro de dez minutos e do prazo de conclusão. Houve recusa 409 de nova autorização após confirmação e após o limite de transferência. | Aprovado no laboratório. |
| Grey box | O WAV válido gerou recibo com estado e confirmação e uma versão no original com o SHA-256 esperado. Com os mesmos bytes declarados Ogg houve `audio-invalid`, zero versões no original e zero fatos de mídia para aquela tentativa. A limpeza removeu duas versões temporárias elegíveis e preservou a versão original confirmada. | Aprovado no laboratório. |
| Grey box | Uma credencial válida no início devolveu cinco respostas 404 equivalentes após a expiração da credencial, nas operações de consulta, autorização, conclusão e replay. | Aprovado no laboratório. |
| Black box guiada | O temporário e o original devolveram 403 a GET anônimo no Moto. O proxy apresentou `no-store`, correlação e `no-referrer` nas respostas verificadas. A página apresentou CSP com origem de upload configurada e bloqueio de enquadramento, `nosniff`, `no-referrer`, `DENY` e `noindex, nofollow`. | Aprovado no laboratório. |
| White e grey box | Os logs da API e do Nginx foram observados após os fluxos. Os 26 marcadores de códigos, autoria, legendas, chaves de tentativa/arquivo, credenciais, consentimentos, assinaturas, versões, referências do armazenamento e checksums ficaram fora dos logs. | Aprovado no laboratório. |

Nenhum achado no recorte verificado.

## Limitações

As verificações locais cobrem os controles da aplicação e o comportamento observável no proxy, no PostgreSQL e no Moto. A autorização e a revogação de novas emissões são decisões da aplicação. As garantias do S3 real sobre o URL já emitido, IAM, KMS, CORS e prova nativa de recebimento precisam do ambiente de homologação, assim como os logs dos serviços de borda e dos coletores AWS. O recorte incluiu observação de segurança defensiva, com volume serial e sem scanner intrusivo. [Estratégia de teste](../spec.md#7-estratégia-de-testes)
