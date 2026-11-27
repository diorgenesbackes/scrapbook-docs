# F10-STORY-001 — Preparação de fotos confirmadas

[Cartão da story](https://trello.com/c/3tAcFFKS) · [Especificação da F10](../spec.md)

- A Outbox encaminha `record.media-confirmed.v1` pela configuração explícita da F10. O consumidor valida a identidade, os vínculos e o original da F08. Confirma a entrega após a transação que admite o trabalho e grava a deduplicação. Os trabalhos de áudio e vídeo permanecem pendentes para as próximas stories.
- Cada réplica tem duas execuções de foto. O lease de dois minutos cobre um orçamento local de 75 segundos. Ao expirar, a transação invalida a tentativa anterior antes de gravar uma nova tentativa e um prefixo exclusivo. Somente a tentativa que mantém a posse pode vincular as versões no banco.
- O subprocesso lê bytes fixados à versão e ao checksum do original. Gera os JPEGs de imagem e miniatura e remove metadados. Grava as duas saídas com criação condicional no S3, confere a integridade por leitura versionada e publica `available` e as referências na mesma transação.
- Falhas transitórias consomem até cinco tentativas. A falha final registra o fato correlacionado para F13. Tanto a projeção de execução quanto o sinal de falha se vinculam ao arquivo, preservando recibo e original. O contrato de leitura é interno à aplicação.

Na revisão unitária da implementação e dos testes, o build concluiu sem warnings e os 432 casos passaram, incluindo 76 novos. A cobertura coletada foi de 53,2% das linhas executáveis alteradas e 89,0% no serviço de preparação. Os adaptadores de banco, mensageria e subprocesso têm exercícios complementares em integração.

A revisão acrescentou reconciliação de publicação SNS com base no recibo de admissão persistido e validação de filas distintas na inicialização. O build final passou com zero warnings e os 447 testes passaram. [Relatório técnico](../code-review/f10-story-001.md)
