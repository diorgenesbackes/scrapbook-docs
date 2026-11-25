# Aceite local do candidato da F08-STORY-002

A execução de 25/11/2026 usou Docker descartável, PostgreSQL, S3 versionado e SQS emulada. Candidatos exercitados `20097b0` (serviço), `c8005e7` (UI), `068add6` (testes), `8d9a753` (infraestrutura) e `ef51987` (Liquibase). [Plano](1-plano.md) · [Cartão](https://trello.com/c/EeNKYoWn)

| Critério | Evidência local |
| --- | --- |
| CA1 | PNG, JPEG e HEIC sintéticos confirmaram após decodificação. Manifesto vazio, excesso de tamanho, legenda de 501 escalares e bytes inválidos foram recusados. Legenda de 500 escalares passou no navegador. Os unitários cobriram também a forma canônica do SHA-256 e os extremos de tamanho e quantidade. |
| CA2 | PUT assinado criou versões privadas na fixture. Renovação e retentativa reutilizaram as credenciais. Os quatro cenários de navegador confirmaram progresso pelo número de bytes, validação separada e recibo genérico. |
| CA3 | Uma prova consumida após a janela confirmou a versão recebida às 13h exatas. Uma prova de 1 ms após o limite resultou em expiração após reconciliação. Nova autorização após prazo e nova tentativa após fechamento retornaram conflito. |
| CA4 | A falha injetada na Outbox preservou o estado aguardando confirmação. O processamento aproveitou a cópia existente na recuperação. Duplicata e PUT tardio preservaram a versão pinada. A limpeza removeu versões temporárias elegíveis e manteve o original e o recibo. |

Passaram 179 testes unitários do serviço, 126 da UI, quatro jornadas Playwright e o runner de integração HTTP/PostgreSQL/Moto e SQS simulada. Os logs da API e da UI omitiram os marcadores privados da fixture. A validação de `dev` ocorre após o merge por PR. A ativação exige comprovação da origem e da versão do evento nativo, KMS, IAM, CORS, proteção do original, limpeza e recursos do decoder em homologação. [E2E](../../test-e2e/f08-story-002.md) · [Segurança](../../test-security/f08-story-002.md)
