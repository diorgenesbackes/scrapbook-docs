# F08-STORY-002 — Tentativa, versão recebida e confirmação da foto

A API admite uma foto JPEG, PNG ou HEIC/HEIF de até 20.000.000 bytes e uma legenda opcional de até 500 escalares. Grava o manifesto, a intenção de armazenamento e a janela admitida no PostgreSQL antes de assinar o PUT. A autorização vincula método, chave, tipo, tamanho e SHA-256, com até dez minutos de validade dentro do prazo de conclusão. Renovação e consulta usam a chave e a credencial da mesma tentativa. [Contratos](../spec.md#5-contratos-da-api)

A evidência aceita vem da fila configurada e inclui o horário UTC do `ObjectCreated:Put` e o `VersionId`. Uma versão recebida às 13h de Brasília pode confirmar após esse horário. O serviço lê exclusivamente a versão fixada, verifica o tamanho e o SHA-256 e decodifica a foto em processo com orçamento de recursos. A cópia exige a mesma versão e o original fica privado. Após falha entre cópia e commit, a recuperação verifica a referência e o hash planejados antes de reutilizar o original.

A confirmação grava os estados de registro, arquivo, sessão e tentativa e seus fatos da Outbox na mesma transação. A limpeza aplica a elegibilidade do recebimento e atua por versão do temporário. Versões sem evidência registrada exigem reconciliação. O recibo contém apenas o resultado e o instante de confirmação. [Prova e reconciliação](../spec.md#3-decisões-técnicas)

`PhotoSubmissions__Enabled` começa falso. A ativação usa `PhotoSubmissions__Bucket`, `PhotoSubmissions__QueueUrl` e `PhotoSubmissions__Region`, a identidade do ambiente e a configuração `Submissions` para concordância. A origem autorizada do bucket é configurada na UI. Endpoints alternativos e path style são exclusivos de Development. [Base de mídia](../dev-infra/f08-story-002.md)
