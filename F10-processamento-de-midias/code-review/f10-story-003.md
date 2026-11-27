# Revisão técnica da F10-STORY-003

A revisão independente comparou o serviço da F10-STORY-003 com `d7a6a4b` e aprovou o candidato `f54095f`. O perfil, a configuração do MediaConvert, a persistência, o estágio autocontido, as permissões herdadas e as regressões locais foram examinados. [Cartão da story](https://trello.com/c/HesFEO9d) · [Especificação](../spec.md)

## Correções verificadas

- O perfil limita dimensões pelo raster orientado, pela razão de pixel e pelos tetos de orientação. A rotação é convertida para a convenção do provedor. Para raster nativo menor que o mínimo de dimensão explícita do MediaConvert, o pedido herda as dimensões. A aceitação desse caso pelo serviço real fica para AWS.
- A sonda calcula a duração da apresentação pelas faixas selecionadas. Um MP4 sintético de 4 s com vídeo e áudio principal de 2 s passou a gerar estágio de 2 s. Um WebM VP9/Opus cujo contêiner declara 300,008 s passou com 299,999 s efetivos. A checagem de pacotes rejeitou remux que perdeu um quadro de 42 ms.
- Fontes rotacionadas que exigem adaptação passaram para H.264 4:2:0 progressivo. O reteste independente confirmou raster, rotação, timestamps, cadência e duração nas amostras pares, entrelaçadas e VFR. A amostra 4:4:4 de 95 × 65 gerou 94 × 64 com SAR 608:611, preservando DAR 19:13, 24 fps, rotação e 48 pacotes. [Requisitos do MediaConvert](https://docs.aws.amazon.com/mediaconvert/latest/ug/auto-rotate-requirements.html)
- A inspeção S3 lê a versão determinada e calcula SHA-256 dos bytes. O HEAD de saída usa as permissões de leitura versionada, preservando o escopo KMS da role. A publicação compara o lease, o arquivo, a tentativa e o perfil antes de atualizar a variante.
- A regressão de áudio cobriu os cinco formatos, a espera do resultado, a reconciliação e a preservação do recibo na falha. Aplicamos uma barreira ao teste unitário concorrente da entrega SQS para que o cenário de indisponibilidade anteceda a associação do job válido.

O build final concluiu sem warnings. O formatter de toda a solução registrou os 24 avisos de espaço em branco em arquivos legados e sugestões informativas. A verificação de espaços nos arquivos C# alterados passou. O coletor `XPlat Code Coverage` ficou indisponível na configuração do repositório. Os 21 testes de infraestrutura e o lint e o build da UI `57f4e63` passaram. [Recursos e configuração](../dev-infra/f10-story-003.md)

O parecer técnico cobre o candidato local. Matriz de entradas, geometrias e cadências efetivas do provedor, transporte EventBridge e IAM em AWS fazem parte do aceite em homologação.
