# Verificação defensiva do depoimento com foto

A revisão usou o serviço `20097b0`, a UI `c8005e7`, a infraestrutura `8d9a753` e o Liquibase `ef51987`. O escopo é a tentativa pública de uma foto, a autorização de escrita e a integridade da versão entregue. [Cartão](https://trello.com/c/EeNKYoWn) · [Contrato](../dev-back/f08-story-002.md)

| Controle | Evidência local |
| --- | --- |
| Autorização limitada | O SDK assinou PUT, host, tipo, tamanho e SHA-256. A tentativa exige evento, chave, segredo e arquivo correspondente. A UI valida a origem exata e os cabeçalhos esperados antes do PUT. |
| Versão e privacidade | O serviço exige `VersionId` no evento S3, verifica o hash da versão fixada e conserva a referência do original confirmado. O recibo contém estado e instante. A limpeza seleciona versões temporárias registradas. |
| Configuração de execução | O template bloqueia acesso público, habilita versionamento e SSE-KMS por ambiente. As permissões delimitam os prefixos e a fila aceita eventos do bucket e da conta previstos. A geração de CSP para Amplify e Nginx foi verificada com origens exatas. |
| Decodificação | Os lockfiles fixam Magick.NET 15.1.1. O processo filho tem orçamento de tempo e memória, diretório privado, ambiente reduzido e saída limitada. A biblioteca limita codificadores, delegates e dimensões. |

A validação do IAM e do KMS efetivos, do bloqueio da leitura pública, da prova nativa S3 → SQS e da recuperação da DLQ depende da AWS em homologação. A fixture Moto usa `VersionId` real do PUT em um envelope sintético para testar o prazo. O processo de decodificação herda a rede, a identidade e as permissões de arquivos do contêiner. A avaliação desse isolamento no runtime publicado é condição de ativação. [Base de mídia](../dev-infra/f08-story-002.md) · [Review](../code-review/f08-story-002.md)
