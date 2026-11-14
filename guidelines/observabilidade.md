# Padrão de logs, métricas e tracing

[Índice](README.md) · [API](api.md) · [Backend](backend.md) · [Mensageria e mídias](mensageria.md) · [HLD](../HLD.md)

**Status:** especificação para implementação. O HLD prevê CloudWatch, OpenTelemetry, X-Ray e alertas SNS por e-mail. Nenhum coletor, dashboard ou alarme foi criado neste trabalho.

Gestão de Eventos implementa a base local em `ActivitySource`/`Meter` `ScrapBook.Events` e JSON Console do .NET, sem exportador externo. O formato atual usa `Timestamp`, `LogLevel`, `EventId` e propriedades nomeadas em `State` (ex.: `Module`, `Operation`, `RouteTemplate`, `CorrelationId`). O futuro coletor deve mapear esses campos ao schema abaixo, sem gerar novo contexto. Catálogo local: 4000 EventsRequestCompleted, 4002 EventsRequestRejected, 4003 EventsUnexpectedFailure. Interrupção do cliente registra `client_aborted` com status ausente, sem inventar HTTP 499.

## 1. Sinais e responsabilidades

PP-STORY-006 reforça a implementação local: logs/spans próprios usam rotas nominais; métodos desconhecidos são OTHER e não labels livres. O Nginx emite somente componente, operação/método categorizados, status e duração. Seu error_log padrão do servidor é suprimido porque inclui URI, mantendo falhas observáveis por status e pelos sinais sanitizados da API. Não registrar Referer, cookies, Authorization ou URLs de evento. Os headers no-referrer reduzem propagação pelo navegador.

O probe manual public_privacy.py inspeciona SQL EF e sinais próprios ScrapBook.Events, sem habilitar coletor externo. Essa evidência não aprova spans automáticos de ASP.NET/Npgsql, baggage nem logs de ALB/WAF/Amplify: configurar redação e repetir os testes antes de habilitar exportação remota. [Contrato e limites](../fdd/FDD-Privacidade-Publica-do-Evento/FDD-Privacidade-Publica-do-Evento.md).

| Sinal | Finalidade | Regra |
| --- | --- | --- |
| Métrica | Volume, taxa, latência, capacidade e atraso. | Agregação, unidade explícita e dimensões limitadas. |
| Log | Contexto seguro de uma execução. | Evento estruturado, classificação e correlação. |
| Trace | Dependências e tempo entre etapas. | Contexto conectado ou relacionado, sem spans duplicados. |
| Auditoria | Alterações sensíveis e autoria. | Persistência e controle de acesso próprios, sem substituir por log amostrado. |

Backend é um serviço único em Clean Architecture, sem módulos: nome lógico proposto `scrapbook-backend`; API, Outbox e consumidores distinguem componente/operação, sem criar serviços fictícios por responsabilidade.

Telemetria não pode ser um caminho alternativo para descobrir conteúdo, remetentes ou quantidade de lembranças bloqueadas. Acesso operacional habitual deve receber sinais técnicos minimizados; correlação com evento específico exige canal restrito e finalidade documentada, sem conceder leitura da mídia.

## 2. Identidade e correlação

| Campo | Significado |
| --- | --- |
| `traceId` | ID W3C do trace, 32 hexadecimais, obtido da Activity. |
| `spanId` | Span ativo, 16 hexadecimais. |
| `requestId` | Identificador local da requisição HTTP. |
| `correlationId` | UUID validado/gerado no ingresso, propagado pela operação e mensagens. |
| `messageId/causationId` | Identidade e causa da mensagem, quando houver processamento assíncrono. |
| `executionId` | Execução de pipeline, migração, reprocessamento ou procedimento operacional. |

Não substituir trace por request ID. Não usar identificador técnico como credencial, chave idempotente ou dimensão de métrica.

Validar contexto W3C externo; cliente não controla coleta ilimitada nem autorização. Sem dados pessoais ou tokens em baggage. Propagar correlação da requisição à Outbox, transporte e consumo. Persistir apenas os metadados necessários, sem URL utilizável.

Consumidores criam span de processamento e seguem a convenção de parent/link apropriada, considerando fan-out, tempo em fila e repetição. Tentativas podem ter spans/traces diferentes mantendo mensagem e correlação. Não presumir um único trace contínuo para dias em DLQ.

## 3. Formato de logs

Eventos JSON, um por linha, com schema/tipos estáveis. Formatter/exporter deve produzir e preservar os campos; usar `ILogger` isoladamente não comprova esse resultado.

| Campo | Regra |
| --- | --- |
| `timestamp` | Instante técnico UTC ISO 8601. Não representa horário de negócio persistido. |
| `severity` | Trace, Debug, Information, Warning, Error ou Critical. |
| `eventId/eventName/message` | Catálogo estável e texto sem payload sensível. `eventId` aqui é código de log, não UUID do evento social. |
| `serviceName/serviceVersion/environment/instanceId` | Serviço, artefato, ambiente e instância. |
| `module/component/operation` | Valores estáveis, como `records`, `consumer`, `media.prepare`. |
| `traceId/spanId/correlationId` | Quando existir contexto. |
| `requestId/httpMethod/route/statusCode` | Em HTTP; rota é template, sem URL concreta/query. |
| `durationMs/outcome` | Duração monotônica e resultado classificado. |
| `errorCode/exceptionType/dependency` | Quando aplicável, sanitizados. |
| `messageId/executionId/attempt` | Somente nos fluxos aplicáveis e canais autorizados. |

Vocabulário de resultado: `success`, `rejected`, `failure`, `timeout`, `client_aborted`, `idempotent_replay`. Request abortado sem resposta não recebe status fictício.

Exemplo alvo sintético:

```json
{
  "timestamp": "2026-11-11T18:00:00.125Z",
  "severity": "Information",
  "eventId": 1000,
  "eventName": "http.request.completed",
  "message": "HTTP request completed",
  "serviceName": "scrapbook-backend",
  "serviceVersion": "1.0.12",
  "environment": "staging",
  "instanceId": "task-example",
  "module": "events",
  "component": "api",
  "operation": "events.public_page.get",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "spanId": "00f067aa0ba902b7",
  "correlationId": "97dfba56-bdf4-49ba-8f64-3cf9e14c134a",
  "requestId": "request-example",
  "httpMethod": "GET",
  "route": "/api/v1/events/{eventId}/public-page",
  "statusCode": 200,
  "durationMs": 85.2,
  "outcome": "success"
}
```

Campos de recurso OTel devem refletir serviço, versão, instância e ambiente. Trace/span usam campos nativos quando exportados. O mapeamento para CloudWatch deve ser testado, sem gerar IDs novos na transformação.

### Emissão, catálogo e níveis

Usar templates estáveis, propriedades estruturadas e catálogo de EventIds. Não serializar objetos de domínio ou mensagens completas. Medir duração por relógio monotônico, sem subtrair horários civis.

Catálogo inicial proposto:

| Faixa / ID | Responsabilidade / evento |
| --- | --- |
| 1000 | `http.request.completed`, um resumo terminal. |
| 1001 | `http.request.aborted`. |
| 1002 | `http.request.failed`, diagnóstico técnico final. |
| 2000–2999 | Falhas de dependências e reconciliação técnica. |
| 3000–3999 | Contas, autenticação e sessões. |
| 4000–4999 | Eventos, convites e falhas de travamento. |
| 5000–5999 | Upload/validação e falhas de processamento. |
| 6000–6999 | Notificações e falhas de encaminhamento. |
| 7000–7999 | Migração e implantação. |
| 8000–8999 | Outbox, consumidores e DLQ. |

Detalhar nomes/IDs antes da implementação e não reutilizar um ID com outro significado. Sucessos individuais de recebimento não devem gerar um feed operacional que revele contagens por evento; evidências persistentes e diagnóstico devem ter acesso compatível com o bloqueio.

Information para resumos e transições permitidas; Warning para degradação recuperável; Error para falha técnica final; Critical para incapacidade de operar/risco concreto de integridade. Debug/Trace temporários e seguros. Não registrar a mesma exceção no repositório, caso de uso e handler.

Reduzir ruído de health bem-sucedido, preservando métricas e transições de falha. Log amostrado não serve como fonte exclusiva de SLI ou auditoria.

### Dados permitidos

Não registrar conteúdo, nome/apelido, legenda, e-mail, senha, cookie, token, chave idempotente, URL assinada/presigned, bodies, query strings, SQL parametrizado com dados, connection string ou mensagem de fila completa. Redigir também spans, headers capturados e logs de SDK.

Não usar nome de arquivo original ou chave S3 com informação pessoal como atributo. IDs do evento/registro/conta só quando necessários em canal protegido, com pseudonimização e finalidade definidas; nunca em assunto de alerta ou dashboard habitual.

Auditoria de mudança de acesso mantém autoria, momento, transição e correlação em armazenamento restrito. Retenção de registros técnicos e auditoria: **180 dias**, conforme PRD/HLD. A configuração deve ser comprovada, inclusive nas exportações e arquivos auxiliares.

## 4. Métricas: nomes, unidades e cardinalidade

Reutilizar instrumentos nativos antes de criar equivalentes. Namespace proposto `scrapbook.*`; duração em **segundos** nos histogramas, milissegundos em `durationMs` dos logs. Confirmar nomes e tipos dos instrumentos nativos na versão adotada.

Exceção explícita do [FDD de Gestão de Eventos](../fdd/FDD-Gestao-de-Eventos/FDD-Gestao-de-Eventos.md#7-observabilidade): suas famílias `scrapbook.events.duration_ms` e `scrapbook.events.dependency.duration_ms` usam `ms`, indicado no nome/unidade. Preservar o contrato dessa primeira instrumentação; converter na integração futura se necessário, sem duplicar a mesma série em unidades diferentes. Não declarar coleta CloudWatch nem retenção de logs locais comprovadas pela existência dos instrumentos.

| Instrumento / visão proposta | Tipo / unidade | Finalidade |
| --- | --- | --- |
| HTTP, banco e dependências nativos | Histogram / `s`, contadores e gauges aplicáveis. | Latência, erros, conexões e saturação. |
| `scrapbook.operation.duration` | Histogram / `s` | Casos de uso por operação/resultado. |
| `scrapbook.upload.attempts` | Counter / `{attempt}` | Tentativas lógicas por resultado limitado. |
| `scrapbook.upload.duration` | Histogram / `s` | Recepção completa, com início/fim definidos. |
| `scrapbook.media.ready.duration` | Histogram / `s` | Confirmação até todas as variantes necessárias estarem prontas, por arquivo/tipo. |
| `scrapbook.media.processing.failures` | Counter / `{failure}` | Falhas finais classificadas. |
| `scrapbook.events.lock.duration` | Histogram / `s` | Primeira confirmação até travamento efetivo. |
| `scrapbook.outbox.pending` | Gauge / `{message}` | Backlog persistente, sem contar a mesma base uma vez por réplica. |
| `scrapbook.outbox.oldest.age` | Gauge / `s` | Idade da pendência mais antiga. |
| `scrapbook.messaging.process.duration` | Histogram / `s` | Processamento por consumidor/resultado. |
| Filas / DLQ | Métricas gerenciadas e reconciliação. | Volume, idade, mensagens em trânsito e falhas retidas. |
| `scrapbook.notification.dispatch.duration` | Histogram / `s` | Momento elegível até aceite pelo serviço de envio. |
| `scrapbook.idempotency.requests` | Counter / `{request}` | Resultado new/replay/conflict/in_progress. |
| Migração / deploy | Resultado e duração por componente. | Evidência de pipeline e recuperação. |

Métricas de upload devem separar **tentativa lógica com erro** de retry de parte HTTP/S3; não disparar o alerta de três envios por três partes do mesmo arquivo. Definir a fonte de verdade e deduplicação da janela de alerta na implementação.

Dimensões permitidas: ambiente, serviço, módulo, operação, template de rota, status, resultado, tipo de mídia, fila/consumidor de catálogo limitado. Sem evento/usuário/registro/message ID, nome, IP, URL, timestamp ou texto livre.

Coletores de estado global não podem duplicar o backlog por número de réplicas. Contadores em memória não são atômicos com o banco: auditoria, confirmação e reconciliação dependem da persistência.

Configurar buckets adequados a cada objetivo: incluir 0,5 s para API, 5 s para travamento e 300 s para mídia, em instrumentos próprios. Agregar histogramas entre réplicas; não calcular média dos percentis individuais. Exportadores podem traduzir nomes/unidades; documentar uma única família por sinal.

Coletar recursos ECS, ALB/WAF, RDS, S3, MediaConvert, SES e custos com instrumentos gerenciados. Nome lógico do pool não pode expor connection string.

## 5. Traces, coleta e retenção

Instrumentar HTTP, PostgreSQL, chamadas AWS, publicação e consumo. Spans com nomes estáveis, sem UUID no nome. Não criar um span por método nem duplicar a mesma consulta em EF e provider.

Falhas tratadas precisam continuar visíveis sem stack/PII irrestritos; verificar o comportamento na versão .NET escolhida. 4xx esperados não devem ser classificados automaticamente como falha interna.

Política de amostragem permanece pendente de volume/custo. Taxa e estratégia serão definidas com evidências da implantação. Se houver head sampling, não prometer preservação de todos os erros.

Testar traces completos nos cenários controlados de validação. Definir buffers, descarte, exportação e flush limitado no encerramento. Falha de telemetria não deve bloquear operações de negócio; perda de coleta precisa ser detectável.

**180 dias se aplicam aos registros técnicos e auditoria conforme o requisito.** Não presumir essa retenção para todo trace ou tipo de métrica no X-Ray/CloudWatch: conferir capacidade do serviço e definir armazenamento/exportação quando necessário, registrando a política efetiva.

## 6. SLIs, dashboards e alertas

| Objetivo | Medição necessária |
| --- | --- |
| Disponibilidade mensal 99,9% | Requisições elegíveis e probes externos, incluindo falhas antes da API. |
| API p95 até 500 ms | Operações comuns sob carga prevista, sem transferência de arquivos. |
| Página pronta em até 3 s | Medição no navegador, perfil 4G e dispositivos documentados. |
| 200 envios simultâneos | Carga de dois eventos com mistura realista de arquivos. |
| 95% das mídias em até 5 min | Confirmação persistente até variante disponível; incluir pendentes/vencidos para não medir só sucessos. |
| Travamento até 5 s | Primeira confirmação até mudança persistente em Eventos. |
| Convite/recuperação até 1 min; abertura até 5 min | Momento elegível até encaminhamento; não confundir com entrega na caixa de entrada. |
| Recuperação até 10 min e RPO zero coberto | Ensaios de falha/restauração e comparação de registros confirmados. |

São metas, não resultados. Definir denominador, janela, volume mínimo, fonte, operações e exclusões antes de declarar SLO atendido.

Disponibilidade deve incluir 5xx/timeouts e saturação atribuível ao serviço; excluir probes do denominador de negócio e rejeições válidas do cliente. Não usar ausência de tráfego/telemetria como sucesso. Não somar o mesmo request medido em ALB e backend.

Dashboards: saúde e versões; latência/erros/carga; ECS e banco; filas/Outbox/DLQ; processamento; e-mails; recuperação; custos. Sem decomposição por evento que revele a surpresa ao responsável.

Alertas obrigatórios:

| Condição | Regra |
| --- | --- |
| Indisponibilidade | Detectar e disparar e-mail em até 2 minutos. |
| Envios com erro | Três tentativas lógicas com erro em janela de 5 minutos, com deduplicação documentada. |
| Processamento definitivo | Alertar falha esgotada, preservando originais. |
| DLQ / publicação | Alertar presença de mensagem retida e falha de Outbox/publicação. |
| Implantação | Alertar falha de pipeline/deploy e identificar versão. |
| Custos | Limiar pendente da definição de orçamento. |

Calibrar alerta de backlog/idade, latência, pool, travamento e falta de telemetria na implantação. Cada alarme tem responsável, severidade, janela, recuperação e procedimento, com controle de repetição. Assinatura SNS do e-mail operacional deve estar confirmada e testada.

Alertas não incluem nomes, conteúdos, URLs ou quantidade de lembranças de um evento. Alerta técnico não concede autorização para abrir arquivos bloqueados.

## 7. Investigação, frontend e pipelines

Consulta ilustrativa para logs JSON no CloudWatch:

```text
fields @timestamp, eventName, operation, statusCode, durationMs, errorCode, traceId
| filter correlationId = "97dfba56-bdf4-49ba-8f64-3cf9e14c134a"
| sort @timestamp asc
| limit 100
```

Selecionar log groups e janela do incidente. Correlacionar versão, resultado, dependência e fila; abrir trace quando retido. Ausência de trace pode decorrer da amostragem, não da ausência de execução.

Frontend mostra mensagem e correlação, sem stack ou captura de formulário/mídia. Eventual coleta de navegador precisa de política própria; não introduzir session replay que capture lembranças.

Liquibase/pipeline registra `executionId`, versão, ambiente, mudanças, duração e resultado. Normalizar saída quando necessário, sem pressupor formato nativo comum. Processo curto precisa entregar evidência antes de terminar; não depender de coleta posterior a um container encerrado.

## 8. Critérios de aceite

- [ ] Erro HTTP pode ser localizado por correlação e associado a trace válido.
- [ ] Mensagem pode ser acompanhada de publicação a consumo/reprocessamento.
- [ ] Logs/spans/exportadores não contêm conteúdo, identidades ou URLs utilizáveis.
- [ ] Dashboards e alertas não revelam metadados de evento bloqueado.
- [ ] Replay não duplica resultado de negócio ou alerta da mesma tentativa.
- [ ] Métricas têm unidades, cardinalidade e agregação verificadas entre réplicas.
- [ ] Pendências/falhas entram na medição das metas, sem viés de sucesso.
- [ ] Alertas de 2 minutos e três envios em 5 minutos foram ensaiados.
- [ ] Retenção de 180 dias, acesso e política real de traces estão documentados e configurados.
- [ ] Falha da coleta é observável sem derrubar o negócio.

Após RF-STORY-001, a propriedade técnica legada module=events permanece no contrato de telemetria para compatibilidade; não representa uma pasta, namespace ou DbContext de módulo. Controllers, serviços e repositórios seguem os conceitos do guideline de backend.
