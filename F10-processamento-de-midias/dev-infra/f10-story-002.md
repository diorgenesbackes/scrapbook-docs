# F10-STORY-002 — Infraestrutura da conversão de áudio

A stack de transcodificação cria uma fila MediaConvert por ambiente. A tarefa pode criar e listar jobs nessa fila, consultar jobs na conta e região e delegar somente a role específica ao MediaConvert. A tarefa grava entradas condicionais e verifica versões. A role do provedor lê entradas e grava saídas em prefixos distintos do bucket versionado, com escopo KMS por objeto e por prefixo. [Template](../../../scrapbook-infrastructure/config/media/media-transcoding.template.json) · [Permissões do MediaConvert](https://docs.aws.amazon.com/service-authorization/latest/reference/list_mediaconvert.html)

A relação de confiança da role exige a fila e a conta da stack. O IAM das duas identidades limita KMS ao S3 regional e aos prefixos aplicáveis. O ajuste foi revisado e retestado na integração em `dev`. [PR #16](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/16) · [Validação defensiva](../test-security/f10-story-002-dev.md)

O EventBridge filtra a conta, região, fila e estados terminais. A fila de resultados separa a entrega da regra das permissões de consumo da tarefa. As falhas de entrega do EventBridge e de processamento pela tarefa usam duas DLQs. [Políticas do EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-use-resource-based.html)

Cada réplica precisa de 1 GiB de espaço temporário gravável para suportar vídeo e áudio juntos. O orçamento considera 500.000.000 bytes para um vídeo F08, 50.000.000 para um áudio F08 e duas preparações F10 de até 114.000.000 bytes cada. O Compose local limita esse espaço com tmpfs. [Configuração e ativação](../../../scrapbook-infrastructure/README.md#conversão-privada-de-áudio)

Os 21 testes de infraestrutura passaram localmente. O aceite em AWS permanece pendente porque a consulta de identidade da AWS CLI retornou `NoCredentials`. Antes de habilitar `MediaAudioProcessing__Enabled`, validar a matriz F08 com entradas e saídas reais, a compatibilidade das chaves de estágio, os filtros, as duas DLQs, as permissões efetivas e o tempo de espera e conversão.

A recuperação manual por redrive é prevista para a fase de recuperação da F10. A política da fila de resultados autoriza a entrega pela regra EventBridge. O redrive nativo exige mudança específica de IAM e verificação em AWS quando a operação for entregue. [Plano da F10](../plan.md#fase-4--entregar-recuperação-estado-e-falhas)
