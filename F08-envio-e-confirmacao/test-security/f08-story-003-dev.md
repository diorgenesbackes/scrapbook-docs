# Verificação defensiva do conjunto de fotos em dev

A revisão independente conferiu o serviço `c8c7ed9`, UI `a46fc57`, testes `81d85ea` e os SHAs da base de mídia no registro da entrega. Foram aprovados 69 unitários filtrados para submissão, fotos, momentos e prova S3, sem falhas ou ignorados. [Entrega](../delivery/f08-story-003.md)

| Controle | Evidência |
| --- | --- |
| Manifesto e ordem | HTTP rejeitou sete manifestos inválidos. Os unitários cobriram alteração da ordem em uma tentativa existente e a recusa por fingerprint divergente. |
| Isolamento | O teste recebeu 404 na rota de outro evento e para chave desconhecida. A consulta exige conjuntamente a tentativa autorizada e a chave do arquivo esperado. O caso de chave válida vinculada a outro registro foi verificado em código. |
| Confirmação | Bucket, chave e versão vinculam a prova. O lock da sessão e a revisão da lease preservam os arquivos reconhecidos e a confirmação integral com os fatos apropriados. |
| Privacidade e limpeza | O status exige credencial, tem `no-store` e preserva somente os dados necessários. O recibo é genérico. A limpeza local removeu versões temporárias elegíveis, os originais foram preservados e os marcadores privados ficaram fora dos logs da aplicação. |

A evidência local usa Moto. A origem S3 nativa, IAM, KMS, CORS, isolamento do decoder publicado e as permissões efetivas de escrita e redrive da DLQ pertencem à homologação. A limpeza local cobre versões com evidência persistida. [Aceite integrado](../test-funcional/2026-11-25_22-38-00/2-execucao.md) · [Redrive](f08-story-002-dev.md#correção-e-reteste)
