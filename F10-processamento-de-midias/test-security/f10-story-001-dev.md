# Validação defensiva da F10-STORY-001 em dev

Aprovados os controles locais de privacidade, escopo de confirmação e integridade verificados. A configuração declarada de IAM, KMS e SNS/SQS passou nos quatro testes de infraestrutura. O isolamento efetivo do subprocesso e as permissões aplicadas na AWS seguem para verificação em homologação. [Cartão F10-STORY-001](https://trello.com/c/3tAcFFKS) · [Spec da F10](../spec.md) · [Revisão técnica](../code-review/f10-story-001.md)

## Candidato e ambiente

| Repositório | SHA revisado |
| --- | --- |
| Serviço | `e9b6fe481d2b1cbd2a2e85fbe2ba6e60aa8dee9d` |
| Testes | `a99d9b681788cd81f14e3c0427992f9febb86fa6` |
| Infraestrutura | `4a2c7e10f5f3b4a3982eab0b4c337a39d3b3f2f1` |
| Liquibase | `8c1f47395a2e6564b6bca8da781c94e5d439a9f3` |
| UI | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| Documentação | `616cdc7359eec66f260040da78cdba2bb73368d3` |

Em 27/11/2026, o laboratório executou o serviço e a UI construídos desses checkouts com PostgreSQL, Liquibase 4.33.0, S3 versionado e SNS/SQS emulados pelo Moto. Docker 29.5.2 em Colima, Python 3.13.15, AWS CLI 2.37.10 apontada ao emulador e FFmpeg/ffprobe 9.0.2 para examinar as variantes. O executor criou um projeto Compose descartável com portas próprias em loopback, usou dois eventos sintéticos independentes para os controles de escopo e descartou contêineres, volumes e massa ao concluir.

## Controles verificados

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| Black box guiada | Três requisições GET anônimas por versão, uma ao original e duas aos derivados. A leitura autenticada no emulador confirmou a existência dos três objetos. | Aprovado localmente. O emulador rejeitou as três leituras anônimas com HTTP 403 ou 404. |
| Black box guiada | Recibo F08 do mesmo recurso com a credencial legítima, com outro bearer e com código de outro evento. | Aprovado. O caso legítimo retornou HTTP 200, `Cache-Control: no-store`, `state` e `confirmedAt`. Os casos alheios foram recusados, sem estado, horário de confirmação ou referências de mídia. |
| Grey box | Confirmação de um JPEG com GPS sintético. Antes da entrega para preparação, a fixture gravou uma versão posterior com bytes e dimensões diferentes sob a mesma chave. Verificou as duas versões do original por SHA-256, leu os derivados pela versão persistida e comparou os respectivos hashes. | Aprovado. O worker produziu os JPEGs de 1.024 × 2.048 e 200 × 400 da versão confirmada. Ambos decodificaram, tiveram orientação aplicada aos pixels e ficaram sem marcadores EXIF, XMP e IPTC. A versão posterior ficou intacta no emulador. |
| Grey box | Entregas SNS com evento divergente para uma identidade conhecida e com ID de mensagem sem fato persistido. O laboratório observou `MEDIA_CONFIRMATION_REJECTED` e conferiu as linhas e versões antes e depois. | Aprovado. Houve um único trabalho e uma única admissão concluída para a foto válida. As variantes prontas permaneceram inalteradas. |
| Grey box | Divergência controlada entre o checksum persistido na fixture descartável e os bytes da versão S3. | Aprovado. A falha correlacionada `MEDIA_SOURCE_INTEGRITY` encerrou o arquivo com zero variantes prontas. O recibo e o original ficaram íntegros. |
| Grey box e white box | Conferência dos logs de serviço e UI com a massa sintética, incluindo legenda, bearer, prova de consentimento, códigos de evento, referências S3, marcadores de URL assinada e envelope canônico. Conferência das chamadas de log do worker e do descarte da saída de erro do subprocesso. | Aprovado para o conjunto examinado. As rejeições registraram o código técnico e os valores protegidos verificados ficaram ausentes dos logs. |
| White box | `MediaAdmissionService`, `MediaConfirmationService` e `MediaPreparationRepository` vinculam confirmação, evento, registro, arquivo e original armazenado. A projeção interna inclui apenas variantes prontas do perfil ativo. `S3MediaVariantStorage` fixa o `VersionId`, verifica a escrita por tamanho e SHA-256 e usa `If-None-Match: *`. | Aprovado na fonte. A autorização da futura galeria pertence ao aceite da F11. |
| White box | `media-processing.template.json` permite leitura por versão nos prefixos de original e derivado, escrita condicional apenas no derivado e uso da chave KMS por S3 com contexto dos dois prefixos. O bucket F08 declara versionamento, bloqueio de acesso público, propriedade dos objetos, TLS e SSE-KMS. | Aprovado na configuração declarada. A política F08 da role da aplicação ainda permite a cópia de `submissions/temporary/` para `submissions/original/` no fluxo de recebimento. Verificar a composição das permissões efetivas em homologação. |
| White box | A assinatura SNS usa entrega raw com filtro pelo tipo `record.media-confirmed.v1`. A fila exige o tópico e a conta configurados, restringe envio direto e declara SSE-SQS, redrive após cinco recebimentos e exclusão da role de execução da DLQ. `MediaProcessingOptions` rejeita o reaproveitamento da fila da F08 ou da F09 ativa. | Aprovado na configuração e na fonte. A entrega e a rejeição pelo consumidor foram observadas no emulador. Redrive e políticas efetivas aguardam AWS. |
| White box | `MediaPreparationWorker` limita a duas operações por réplica. `MediaPreparationService` limita a entrada a 20 MB, a tentativa a 75 s e o processamento a cinco tentativas com backoff. O runner fixa perfil 2.048/400/85 e rejeita tamanho fora do intervalo. O subprocesso tem timeout de 20 s, encerramento da árvore, trabalho em diretório temporário e ambiente reduzido. `MagickPhotoDecoder` limita formatos, frames, dimensões, pixels, memória, perfil, disco e tempo. `PhotoVariantCodec` limita cada saída a 6 MiB. | Aprovado na fonte para limites e interrupção. Os casos válidos executaram o subprocesso local. A análise de contenção verificou a configuração de limites, backoff e encerramento das tentativas. A pressão sobre recursos é medida no ensaio de carga. |

## Limitação herdada e verificações em homologação

A política do ImageMagick desabilita delegados e filtros, bloqueia caminhos `@*` e permite somente codificadores de foto. O Compose usa filesystem raiz somente leitura, `/tmp` em tmpfs, remoção de capabilities e `no-new-privileges`. O subprocesso compartilha a identidade de rede e filesystem do serviço, sem namespace de rede ou conta do sistema próprios. A verificação de conformidade com o requisito de filesystem limitado e ausência de rede da spec fica pendente na tarefa efetivamente implantada em homologação. [Spec da F10](../spec.md#identidade-privacidade-e-reconciliação) · [Revisão técnica](../code-review/f10-story-001.md)

A validação em homologação deve conferir a role efetiva da tarefa e a política aplicada no bucket, KMS, tópico e filas. Ela deve cobrir leitura versionada permitida, leitura anônima recusada, escrita condicional, trânsito SNS → SQS da conta e DLQ após o limite de recebimentos.

## Comandos e resultados

```sh
DOCKER_CONTEXT=colima-scrapbook-f08-validation TMPDIR=/private/tmp python3 -B f1001_security_check.py
python3 -B -m unittest discover -s tests -p 'test_media_processing.py' -v
```

O primeiro comando executou um teste dirigido com os grupos acima em 65,216 s, código 0. O script temporário reutilizou os helpers do checkout de `scrapbook-tests`. O segundo, executado em `scrapbook-infrastructure`, teve quatro testes aprovados e código 0. O restore adicional de unitários no host foi interrompido porque o ensaio específico já havia concluído e a máquina seria reservada para carga. [Saída sanitizada do ensaio](ensaio-local.txt) · [Testes de infraestrutura](politicas-iac.txt)
