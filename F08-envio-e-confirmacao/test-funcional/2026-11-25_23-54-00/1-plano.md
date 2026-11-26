# Plano de aceite local da F08-STORY-004

O ensaio usa dois eventos, convidados e vídeos sintéticos, PostgreSQL 18.6 e um bucket versionado descartável. O relógio da API de teste permite observar a conclusão às 13h de Brasília e um milissegundo depois. A fixture envia uma notificação com o VersionId devolvido pelo armazenamento local. [Story e critérios](https://trello.com/c/UR7dLziQ) · [Spec](../../spec.md)

| Critério | Cenário | Resultado esperado |
| --- | --- | --- |
| CA1 | Seleção MP4 e WebM no momento e MP4 e MOV no depoimento, captura de vídeo e vídeos de 60 e 300 s. | Um arquivo compatível passa e recebe recibo. Formato, quantidade, duração ou tamanho fora do contrato são recusados. |
| CA2 | Perder a segunda parte depois de concluir a primeira e consultar a mesma tentativa. | A retentativa envia somente as partes pendentes. S26 inicia o progresso retomado com os bytes confirmados. |
| CA3 | Concluir multipart, atrasar a notificação e consultar o estado antes de recebê-la. Repetir com uma notificação de conclusão depois do prazo. | A versão tempestiva pode ser validada após as 13h. A versão comprovadamente tardia é rejeitada. |
| CA4 | Forçar falha de commit depois da criação do original, repetir a validação e cancelar um multipart incompleto elegível. | A aplicação reaproveita a versão privada sem recibo antecipado. A limpeza preserva o original e remove somente temporários próprios. |

O comando local é `python3 -B video_submission_stack.py --with-browser` em `scrapbook-tests` com as revisões correspondentes de serviço, UI, infraestrutura e Liquibase. Os cenários de carga e segurança têm registros próprios. IAM, KMS e notificações nativas do S3 serão exercitados em homologação antes da ativação pública. [Infraestrutura](../../dev-infra/f08-story-004.md)
