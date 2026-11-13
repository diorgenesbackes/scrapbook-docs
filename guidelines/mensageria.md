# Padrão de mensageria, upload e processamento de mídias

[Índice](README.md) · [HLD](../HLD.md) · [Backend](backend.md) · [API](api.md) · [Observabilidade](observabilidade.md)

Este guia complementa a estrutura herdada do Expenses, cuja execução de negócio é síncrona. O Scrap Book possui processos assíncronos aprovados, com API e consumidores no mesmo backend. Parâmetros operacionais ainda abertos devem ser definidos e medidos antes da publicação.

## 1. Responsabilidades e entrega

- Cada fluxo explicita seus contratos de mensagem, Outbox, deduplicação e efeitos persistentes; as classes ficam agrupadas por conceito no serviço único.
- Eventos internos são distribuídos pelo SNS às filas dos consumidores interessados; trabalhos específicos podem usar SQS diretamente conforme o contrato.
- Usar SQS Standard, admitindo mensagens repetidas e fora de ordem. Não depender de entrega exatamente uma vez.
- Manter DLQ por fila, concorrência limitada e recuperação documentada.
- Filas não substituem registros confirmados no PostgreSQL e arquivos no S3.
- Não criar serviço genérico de workers separado; API, publicador e consumidores compartilham o serviço ECS.

Interações que precisam de resposta imediata usam contratos internos. O travamento após o primeiro registro é explicitamente eventual, com meta de até 5 segundos em condições normais. Não prometer atomicidade entre processos assíncronos para eliminar essa janela sem nova decisão.

## 2. Envelope de mensagem

Proposta de formato, a formalizar nos contratos de cada fluxo:

```json
{
  "messageId": "00000000-0000-0000-0000-000000000201",
  "type": "records.record_confirmed",
  "version": 1,
  "occurredAt": "2026-11-11T18:00:00Z",
  "eventId": "200",
  "correlationId": "97dfba56-bdf4-49ba-8f64-3cf9e14c134a",
  "data": {
    "recordId": "202"
  }
}
```

`eventId` identifica o evento social do Scrap Book; `messageId` identifica a ocorrência de mensageria. IDs relacionais expostos são strings decimais; `messageId` e `correlationId` permanecem UUIDs técnicos e não substituem a PK inteira de uma tabela Outbox ou de deduplicação. Usar `causationId` e contexto W3C em atributos/envelope quando aplicáveis, com convenção única e instrumentação testada.

Transportar referências mínimas, sem bytes, texto de depoimento, nome do convidado, e-mail, tokens ou URLs utilizáveis. O consumidor consulta os dados necessários pela porta autorizada. Contratos de notificações devem resolver o destinatário em armazenamento protegido.

Republicar a mesma mensagem mantém `messageId`. Mensagem nova derivada recebe outro identificador e referencia a causa. Adições opcionais devem ser toleradas; mudança incompatível exige versão nova e plano para mensagens antigas em fila/DLQ.

## 3. Outbox transacional local

1. Validar o caso de uso no serviço e abrir sua transação.
2. Persistir alteração, auditoria pertinente e mensagem de Outbox na mesma unidade.
3. Confirmar a transação; não depender de publicação bem-sucedida para preservar a intenção.
4. Publicador reivindica trabalho de forma persistente e segura entre réplicas.
5. Publicar e registrar conclusão; se houver falha antes de registrar, admitir republicação.

Claims/leases precisam expirar ou ser recuperáveis após queda. A estratégia SQL, índices e prazo serão definidos no modelo físico. Não marcar como publicada antes de o transporte aceitar a mensagem.

Não manter transação/lock aberto durante chamada de rede. Medir backlog e idade da Outbox. Limpeza só após política de retenção e recuperação definida; não apagar pendências para encerrar alarmes.

## 4. Consumidores, deduplicação e falhas

Cada processamento deve:

- Validar tipo, versão, tamanho e campos antes de executar.
- Criar escopo DI e contexto próprios; respeitar o limite de concorrência por tipo de trabalho.
- Deduplicar persistentemente por consumidor e `messageId`, com unicidade sob disputa entre réplicas.
- Confirmar deduplicação e efeito local juntos. Reserva sem conclusão precisa ser recuperável.
- Excluir/confirmar a mensagem no transporte somente após conclusão durável.
- Tratar repetição e evento antigo sem regredir estado nem duplicar efeitos.
- Renovar visibilidade quando necessário e parar novas buscas no encerramento.

Definir por fila: timeout e renovação de visibilidade, concorrência por tarefa, máximo de tentativas, backoff, retenção, DLQ e redrive. Esses números estão pendentes no HLD; não copiar defaults como se atendessem às metas.

Falha transitória permite nova tentativa limitada. Mensagem inválida/incompatível ou falha esgotada exige quarentena/DLQ e alerta. Reprocessamento deve registrar causa, revisão corretiva, seleção, execução e resultado, preservando a identidade lógica para não duplicar negócio.

Efeito em SES, S3 ou MediaConvert não participa da transação PostgreSQL. Persistir intenção, referência externa e estado; usar mecanismos de idempotência disponíveis e reconciliação. Queda entre efeito externo e gravação pode deixar resultado desconhecido; deduplicação local isolada não garante um único e-mail ou trabalho externo.

## 5. Início, prazo e transferência de upload

O início elegível ocorre quando o backend recebe a solicitação de sessão após o convidado tocar em Enviar, com revisão concluída e chave idempotente. Selecionar arquivo ou começar gravação não reserva prazo.

| Marco | Regra |
| --- | --- |
| Abertura | Duas horas antes do evento. |
| Novos envios | Encerram ao meio-dia do dia seguinte, em Brasília. |
| Conclusão / repetição elegível | Até as 13h, para tentativa iniciada no prazo. |
| Processamento posterior | Permitido quando todos os arquivos foram recebidos integralmente dentro do prazo. |
| Incompletos / rejeitados | Limpeza após 24 horas; mecanismo e atraso operacional a validar. |

Definir precisamente as comparações inclusivas/exclusivas de cada fronteira na FDD e testar imediatamente antes, no marco e após. O horário do navegador não comprova início nem conclusão.

Persistir arquivos esperados e evidências confiáveis de recepção no servidor/armazenamento. Validação atrasada não deve rejeitar como tardio um arquivo integralmente recebido a tempo. Uma notificação de S3 atrasada não pode ser confundida com o instante real de recepção.

Autorizações temporárias devem restringir objeto, operação, tamanho e demais condições suportadas. Expiração e reconciliação devem respeitar a tolerância de negócio; testar transferências iniciadas antes e concluídas depois do prazo, sem confiar apenas na validade da URL.

Objetos usam chaves geradas pelo servidor, sem nomes pessoais e sem possibilidade de substituir original confirmado. Upload não concede `GetObject` ou listagem ao convidado. Vídeos grandes usam multipart com repetição das partes falhas; não transferir tudo outra vez por padrão.

## 6. Validação e confirmação

Validar conteúdo efetivo, não somente extensão, MIME declarado ou duração informada pelo navegador:

| Categoria / formato | Quantidade | Limites de produto |
| --- | --- | --- |
| Depoimento em foto | Uma foto | 20 MB. |
| Depoimento em vídeo | Um vídeo | 500 MB e 5 minutos. |
| Depoimento em áudio | Um áudio | 50 MB e 5 minutos. |
| Depoimento em texto | Um texto | 5.000 caracteres. |
| Momento em fotos | De uma a 10 fotos | 20 MB por foto; conjunto indivisível. |
| Momento em vídeo | Um vídeo | 500 MB e 1 minuto. |
| Legenda de mídia | Uma legenda opcional por envio | 500 caracteres; comum a todas as fotos do conjunto. |

Rejeitar arquivos vazios, corrompidos e incompatíveis. Não combinar formatos no mesmo depoimento nem adicionar corte/edição para adequar conteúdo.

O fluxo deve separar recepção, validação, promoção do original e confirmação:

1. Receber na área temporária privada e registrar evidências.
2. Validar todos os arquivos esperados e dados obrigatórios.
3. Assegurar a persistência dos originais e metadados necessários.
4. Confirmar o registro e sua Outbox na transação da operação.
5. Responder/atualizar status como confirmado e agendar preparação.

S3 e banco não têm commit conjunto. A implementação deve reconciliar queda entre cópia/persistência de objetos e commit local, sem confirmar referências ausentes. Para conjuntos, nenhuma foto entra na galeria como recebida antes da confirmação integral. Objetos órfãos precisam de limpeza segura; não apagar arquivos de um registro confirmado.

Estados conceituais do HLD: recebendo, validando, confirmado, processando, disponível, rejeitado e falha de processamento. Registro, arquivo e trabalho podem ter estados próprios. Confirmação deve permanecer persistida independentemente da disponibilidade das variantes; falha de conversão não transforma registro confirmado em upload rejeitado.

## 7. Preparação e preservação

| Entrada prevista no HLD | Preparação |
| --- | --- |
| JPEG, PNG, HEIC/HEIF | JPEG até 2.048 px no maior lado; miniatura até 400 px. |
| MP4, MOV, WebM | MP4/H.264, áudio AAC, até 1080p e 30 fps. |
| MP3, M4A, WAV, OGG, WebM | M4A/AAC. |

Preservar orientação, proporção, resoluções e taxas menores, sem ampliação. Remover localização das fotos exibidas. Bibliotecas, codecs efetivos, bitrates e vídeo vertical precisam de testes; extensão na lista não comprova decodificação.

Fotos são processadas no consumidor .NET; vídeos e áudios no MediaConvert. Resultados chegam por EventBridge/SQS. Persistir trabalho e referência externa antes de perder a capacidade de reconciliar; eventos repetidos/atrasados não podem reabrir um trabalho concluído nem sobrescrever uma variante válida.

Manter originais privados mesmo após falha definitiva. Não servir original como fallback que contorne autorização ou exponha localização. A meta é 95% das mídias prontas em até 5 minutos após confirmação, sob perfil de carga documentado.

## 8. Travamento e notificações

`record_confirmed` desencadeia travamento em Eventos e Convites. A transição deve ser idempotente e não revelar ao responsável contagens ou remetentes por telemetria. A janela eventual de até 5 segundos é meta operacional aceita; medir da primeira confirmação ao travamento efetivo.

Scheduler diário às 00h de Brasília envia trabalho para localizar eventos liberados com aviso pendente. A consulta de acesso usa a data em cada pedido, sem depender do job.

Controlar notificações e tentativas para evitar duplicação por varredura repetida ou concorrente. Convites e recuperação têm meta de encaminhamento em até 1 minuto; abertura, até 5 minutos. Recuperação segue o fluxo Cognito aprovado, sem recriar link próprio de 30 minutos. A forma de envio e evidência da integração Cognito/SES devem ser validadas.

## 9. Critérios de aceite

- [ ] Queda entre commit/publicação e efeito/ACK é recuperável sem novo registro.
- [ ] Duplicação, mensagens fora de ordem, versão desconhecida e múltiplas réplicas foram testadas.
- [ ] Visibilidade, shutdown, DLQ e redrive têm parâmetros e procedimentos validados.
- [ ] Primeiro travamento e avisos têm medição e proteção contra efeitos repetidos.
- [ ] Prazo usa evidência de recepção, inclusive quando validação demora.
- [ ] Conjunto parcial não confirma; originais confirmados sobrevivem à falha de processamento.
- [ ] Limpeza após 24 horas não alcança arquivos confirmados e trata multipart incompleto.
- [ ] Testes locais simulados são complementados por integrações reais em homologação.
- [ ] Carga combinada de API, consumidores e mídia sustenta as metas antes do uso no evento.
