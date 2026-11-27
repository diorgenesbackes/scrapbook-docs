# F10-STORY-003 — Entrega para homologação

A F10-STORY-003 prepara o vídeo confirmado pela F08 para reprodução privada em MP4 H.264, com AAC-LC quando há áudio. Mantém a orientação e a proporção de exibição, limita a resolução e a cadência e preserva a versão original. [Cartão no Trello](https://trello.com/c/HesFEO9d) · [Especificação](../spec.md)

| Componente | Pull request em dev | SHA integrado |
| --- | --- | --- |
| Serviço | [#21](https://github.com/diorgenesbackes/scrapbook-service/pull/21) | `78abe427054ee420cab27da00b1a772534e390b9` |
| Infraestrutura | [#18](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/18) | `1578ca0db337c58b0ce546e4ea252b89527443f1` |
| Testes | [#20](https://github.com/diorgenesbackes/scrapbook-tests/pull/20) | `e19c13af9726e5605b3f7595e6a7d0637e11ecd1` |
| Documentação inicial | [#29](https://github.com/diorgenesbackes/scrapbook-docs/pull/29) | `99044c55707eeab1f0539e5adab523d19e335734` |

A entrega usa a F08 integrada, a admissão e o schema da F10-STORY-001 e a fila e a orquestração da F10-STORY-002. O candidato usa as colunas de vídeo existentes. O estado dos PRs de promoção e o resultado da publicação ficam registrados no cartão. [Cartão no Trello](https://trello.com/c/HesFEO9d)

## Evidências locais

- Build sem warnings, 667 testes unitários do serviço, 21 verificações de infraestrutura e revisão técnica independente. [Relatório de revisão](../code-review/f10-story-003.md)
- A regressão de áudio verificou MP3, M4A, WAV, Ogg e WebM, além de espera do provedor, reconciliação e falha de verificação. [Contrato compartilhado](../dev-back/f10-story-003.md)
- Oito perfis de vídeo, três cenários assíncronos e duas jornadas Chromium passaram no candidato integrado. [E2E](../test-e2e/f10-story-003-dev.md) · [Aceite funcional](../test-funcional/2026-11-27_15-09-45/2-execucao.md)
- Os 18 vídeos da rajada concorrente ficaram prontos nos dois eventos. O p95 da confirmação até a observação da prontidão foi 1,17 s. O ensaio usou uma fixture de contrato do MediaConvert. [Carga local](../test-performance/f10-story-003/f10-story-003-dev.md)
- A validação defensiva confirmou a recusa de seis leituras anônimas e quatro consultas cruzadas de recibo. As versões preservaram seus hashes, e a variante validada da amostra georreferenciada saiu sem a tag de localização. [Segurança defensiva](../test-security/f10-story-003-dev.md)

## Aceite em homologação

A consulta da identidade AWS retornou `NoCredentials`. Valide a matriz de entradas, a codificação H.264/AAC real, a ausência de áudio em vídeos mudos, rotação, aspecto, escala, cadência, duração, cor e remoção de localização. Inclua fontes MOV e MP4 autocontidas, WebM, múltiplas faixas, VFR, fontes com crominância 4:4:4 ou entrelaçadas que precisem de adaptação e o comportamento quando a saída herda dimensões pequenas. Confira o limite de preparação de arquivos grandes no ambiente alvo. [Perfil e limites](../dev-back/f10-story-003.md)

Confira as versões privadas em S3, a relação de confiança da role, KMS, entrega EventBridge/SQS e as duas DLQs. A flag `MediaAudioProcessing__VideoEnabled` começa desabilitada. A ativação em homologação exige a matriz da F08 e as permissões efetivas verificadas. A tarefa precisa de pelo menos 3 GiB no diretório temporário gravável. Em pausa, desabilite a flag e preserve versões, filas e tentativas. [Infraestrutura](../dev-infra/f10-story-003.md)
