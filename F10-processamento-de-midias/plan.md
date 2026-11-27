# Plano de implementação da F10 — Processamento de mídias

O trabalho abrange serviço, Liquibase, infraestrutura, testes e documentação. Cada fase inclui a verificação da capacidade que entrega. [Especificação da F10](spec.md)

## Pré-requisitos

- Usar como base a F08 integrada, com confirmação transacional, Outbox por arquivo e armazenamento privado versionado. Verificar o fato emitido e os formatos de mídia aceitos na versão que servirá de base. [Spec da F08](../F08-envio-e-confirmacao/spec.md)
- Conferir o esquema da Onda 7 e as permissões das roles, junto aos recursos de SNS/SQS de referência da F09. A execução local usa PostgreSQL, armazenamento e mensageria emulados, JPEGs reais e um adaptador de contrato para o MediaConvert. [Dados da F10](spec.md#6-modelo-de-dados) · [Mensageria da F09](../F09-travamento-apos-primeiro-registro/spec.md)
- Para o aceite em homologação, disponibilizar conta, região, fila e quota para MediaConvert, S3 versionado, IAM e EventBridge/SQS. Registrar a configuração e a identidade operacional autorizada para reprocessamento. [HLD](../HLD.md)

## Fase 1 — Admitir arquivos confirmados

1. Conferir as constraints e índices existentes. Aplicar o changeset aditivo de contexto da confirmação e relógio de submissão, com validação em bases nova e atualizada.
2. Provisionar a assinatura por tipo, fila de confirmação e DLQ. Habilitar o roteamento do fato da F08 com configuração independente, preservando o comportamento do fluxo da F09.
3. Implementar admissão idempotente e escopada ao arquivo confirmado, com trabalho e variantes pendentes gravados junto com a deduplicação.
4. Provar confirmação do recebimento antes do processamento, admissão única com duas réplicas, reentrega, quarentena e ausência de trabalhos para texto e conjuntos rejeitados.

## Fase 2 — Gerar e preparar as variantes de foto

5. Implementar reivindicação de trabalho, lease e persistência das tentativas com contexto e prefixos privados.
6. Gerar JPEG principal e miniatura com processador isolado, limites de recursos, orientação, proporção, remoção de localização e leitura da versão confirmada do original.
7. Verificar os dois JPEGs e gravar a disponibilidade de forma atômica. Testar retomada entre gravação dos objetos e commit, arquivo pequeno, foto rotacionada e metadados GPS.

## Fase 3 — Produzir áudio e vídeo com o MediaConvert

8. Fechar a matriz de entradas F08, perfis e formatos reais com artefatos sintéticos. Preparar o caminho de compatibilidade para entradas aceitas que exigirem normalização.
9. Configurar o MediaConvert, os prefixos e roles privados, a entrega de resultado EventBridge/SQS e as respectivas DLQs. Validar as configurações no adaptador local e no ambiente AWS.
10. Implementar entrada imutável por tentativa, submissão com token persistente e aplicação de resultados. Exercitar callback adiantado, resposta perdida e retries dentro e fora da janela idempotente.
11. Verificar e promover MP4 H.264/AAC e M4A/AAC. Registrar evidências de rotação, tamanho, taxa de quadros, proporção e compatibilidade de entradas em homologação.

## Fase 4 — Entregar recuperação, estado e falhas

12. Reconciliar leases expirados e resultados ausentes ou divergentes. Testar eventos fora de ordem, falhas transitórias, tentativas esgotadas e jobs de resultado incerto.
13. Expor no contrato interno o estado e as referências prontas para a F11. Gravar falhas definitivas para a F13 com identidade por ciclo. Conferir correlação, integridade do recibo e separação por evento e arquivo.
14. Implementar histórico em lotes, redrive auditado e reabertura de um ciclo após correção. Validar retomada, retenção da proteção idempotente, prioridade de envios novos e limites da limpeza versionada.

## Fase 5 — Preparar a ativação em homologação

15. Instrumentar os indicadores de fila, prontidão, falha, reconciliação e uso de recursos. Documentar parâmetros, permissões e recuperação sem expor referências privadas.
16. Executar o perfil misto de carga sobre o candidato, comprovar a meta no ambiente AWS e registrar os resultados por tipo, incluindo impacto na confirmação da F08 e comportamento na rodada de borda.
17. Validar pausa, retomada e rollback operacional. Ativar com capacidade gradual, reconciliar o histórico em lotes e monitorar backlog, prontidão e falhas segundo o perfil aceito.
