# Guideline de backend

[Índice](README.md) · [Princípios](principios.md) · [Contratos HTTP](api.md) · [Mensageria e mídias](mensageria.md) · [Observabilidade](observabilidade.md)

Aplicação: C#/.NET e ASP.NET Core 10. SDK 10.0.401, runtime validado 10.0.12, solution `ScrapBook.sln` e xUnit 2.9.3. A feature Gestão de Eventos fixa EF Core 10.0.15, Npgsql/provider 10.0.3 e Microsoft.AspNetCore.OpenApi 10.0.15, com lockfiles. Consulte o [backend na feature](https://github.com/diorgenesbackes/scrapbook-service/blob/feature/GE-STORY-001/README.md). Integrações AWS permanecem pendentes; essa branch não representa publicação.

Revisão normativa do responsável em **2026-11-15**, para o reinício do projeto: **regras de negócio existem somente nas Services de Application**; repositórios executam persistência; as demais classes oferecem contratos, dados ou operações utilitárias/técnicas. Código sem comentários, exceptions próprias e tratamento global na API são obrigatórios. Application e Infrastructure não se referenciam em nenhuma direção. Esta revisão substitui as orientações anteriores que permitiam políticas de negócio em Domain e portas em Application; não declara o código existente refatorado.

## 1. Clean Architecture por conceito

Decisão do responsável em **2026-11-13**: serviço único organizado por camadas e conceitos, registrado no [ADR-001](../adr/ADR-001-backend-clean-architecture.md). Retirar Modules, Endpoints e subpastas funcionais. A API utiliza controllers MVC reais, derivados de ControllerBase, com ApiController e rotas por atributos; composição por AddControllers/MapControllers.

| Responsabilidade | Conteúdo | Dependências permitidas |
| --- | --- | --- |
| Domain | Entidades POCO, contratos de dados, interfaces/portas, exceptions próprias e utilitários sem decisões de negócio. | Somente biblioteca base do .NET; sem pacotes externos nem referências a outros projetos. |
| Application | Services: regras de negócio, autorização do caso de uso, validações, transições e coordenação de operações. | Somente Domain. Sem ASP.NET, EF Core ou SDKs de integração. |
| Infrastructure | Repositórios, EF Core e adaptadores Cognito/S3/SQS/SNS/SES/MediaConvert, implementando portas de Domain. | Somente Domain entre os projetos; pacotes técnicos necessários aos adaptadores. |
| API / host | Controllers, adaptação HTTP/MVC/OpenAPI, proteção de dados, GlobalExceptionHandler e composição. | Application, Infrastructure e Domain, com referências diretas explícitas. Infrastructure é usada na composição. |
| UnitTests | Testes unitários com dependências substituídas. | Unidades sob teste. |
| Repositório de testes | Integração, HTTP, banco, navegador, carga, segurança e recuperação. | Componentes necessários ao cenário, por acionamento manual. |

### Referências obrigatórias entre projetos

As setas representam `ProjectReference`, sem dependência recíproca:

```text
API            -> Application, Infrastructure, Domain
Application    -> Domain
Infrastructure -> Domain
Domain         -> nenhum projeto
```

**É proibido Application referenciar Infrastructure e Infrastructure referenciar Application**, inclusive para interfaces, exceptions, commands, telemetria ou resolução indireta de serviços. Não contornar esse limite com reflexão, `dynamic`, Service Locator ou callbacks que entreguem decisões de negócio a um repositório.

Interfaces compartilhadas, como IEventService, IEventRepository, IUnitOfWork e portas de integração/telemetria, ficam em Domain/Interfaces. Commands, requests, responses, modelos e exceptions usados entre camadas também ficam em Domain. Application implementa as Services; Infrastructure implementa persistência e integrações; API registra as implementações em Program.cs/extensões de composição. Nenhuma implementação de Infrastructure recebe uma Service de Application.

Agrupar todas as classes do mesmo conceito no diretório correspondente da camada, por exemplo API/Controllers, Application/Services e Infrastructure/Repositories. Não criar Services/Events, Repositories/Identity ou divisões funcionais equivalentes.

Os cinco diretórios de projeto ficam diretamente na raiz de scrapbook-service, junto de ScrapBook.sln. Não criar intermediários src/test/tests, pastas virtuais correspondentes nem diretórios vazios de reserva.

```text
scrapbook-service/
  ScrapBook.sln
  ScrapBook.Api/
  ScrapBook.Application/
  ScrapBook.Domain/
  ScrapBook.Infrastructure/
  ScrapBook.UnitTests/
```

Executar a API a partir dessa raiz com `dotnet run --project ScrapBook.Api --launch-profile http`. Referências entre projetos usam os diretórios irmãos; o projeto de testes unitários segue a mesma organização dos demais.

| Camada | Diretórios por conceito |
| --- | --- |
| API | Controllers, Workers, ExceptionHandlers, Middlewares, Validators, Options, Readers, Writers, InputFormatters, Transformers, DataProtectors, Extensions |
| Application | Services |
| Domain | Interfaces, Entities, Commands, Models, Requests, Responses, Mappers, Readers, Builders, Exceptions, Enums, Clocks, Scopes |
| Infrastructure | Repositories, DbContexts, Configurations, Mappers, UnitOfWorks, Extensions |
| UnitTests | Conceito testado, com doubles em Stubs e providers em Providers |

EventController depende de IEventService; EventService utiliza IEventRepository e IUnitOfWork, todos com contratos em Domain. Infraestrutura implementa repositórios e um ScrapBookDbContext scoped, compartilhado na requisição, com IEntityTypeConfiguration separado por entidade. Os schemas físicos identity/events/records/notifications e as chaves existentes de configuração/telemetria não definem módulos de código e continuam compatíveis.

Operação local e Outbox, quando aplicável, compartilham transação curta. Integrações externas não participam de locks/transações do banco. Liquibase permanece o executor exclusivo de DDL; startup não migra ou prepara dados.

### Contratos puros e adaptadores técnicos

Domain concentra contratos e utilitários independentes de frameworks, usando somente a biblioteca base do .NET. **Comportamento puro não é autorização para colocar regras de negócio em Domain.** Se uma operação decide elegibilidade, limite, prazo, permissão, transição ou efeito de negócio, ela pertence a uma Service de Application. Não adicionar dependências para viabilizar uma movimentação nem criar diretórios vazios para conceitos ainda inexistentes.

- Requests/Responses são records de dados em Domain; atributos de System.Text.Json, parte da biblioteca base, são permitidos. HttpContext, binding MVC, headers, status, ProblemDetails e OpenAPI ficam na API. Não serializar entidades rastreadas como resposta.
- JsonBodyReader recebe Stream, Type e CancellationToken, sem conhecer HTTP. API/Readers/HttpJsonBodyReader verifica mídia/limite técnico do body e lança exceptions próprias de entrada; EventMvcJsonInputFormatter conecta esse adaptador ao MVC. HealthHttpResponseWriter escreve a resposta HTTP; EventOpenApiDocumentTransformer adapta o documento OpenAPI.
- Mappers e builders apenas copiam/convertem dados fornecidos. Um EventAuditBuilder pode montar o formato do registro; a Service decide se, quando e com quais fatos auditar. EventResponseMapper não decide ocultação ou acesso. Infrastructure/Mappers/EfPostgresExceptionMapper traduz exceções EF/Npgsql para exceptions técnicas definidas em Domain, preservando a causa; não importa Application nem decide falhas de negócio.
- Entities contém EventEntity e os POCOs com sufixo PersistenceEntity. Representam dados, sem atributos EF, geração de schema, validações de negócio ou transições próprias. DbContext, tracking, IEntityTypeConfiguration, locks e comparação atômica da versão permanecem em Infrastructure; a Service decide o significado dos resultados.
- **Services existem exclusivamente em Application**, inclusive EventService, EventTelemetryService, EventCursorService e DevelopmentCurrentActorService. Dependem de portas quando precisam de recursos externos. EventCursorService valida payload/conta/validade do cursor pela porta IEventCursorProtector; API/DataProtectors/EventCursorDataProtector implementa a proteção ASP.NET. Manter propósito, chaves e duração compatíveis. Application também não referencia ASP.NET ou EF.

Nomes técnicos devem explicitar protocolo/framework e terminar no conceito (HttpJsonBodyReader, EventMvcJsonInputFormatter, HealthHttpResponseWriter, EfPostgresExceptionMapper, EventCursorDataProtector). Interfaces mantêm I e a responsabilidade, como IEventCursorProtector. Evitar serviços técnicos chamados Service fora de Application.

## 2. Fluxo de um caso de uso

O controller recebe entrada, identidade e cancelamento; chama a Service e adapta o sucesso para HTTP. A Service autoriza, consulta portas de Domain, valida todas as regras, coordena a transação, decide os efeitos e retorna dados. Infrastructure executa I/O solicitado; Domain fornece dados, contratos e utilitários. Exceptions seguem para GlobalExceptionHandler na API.

### Regras de negócio somente nas Services

**DEVE:** colocar nas Services todas as decisões de vínculo, autorização por evento, janela de envio/abertura, limites do produto, campos travados, versão esperada, idempotência, transições, auditoria e criação/cancelamento de Outbox ou notificações. Dividir o fluxo em métodos pequenos e, quando houver responsabilidades distintas, Services coesas.

**NÃO DEVE:** colocar essas decisões em repositories, controllers, handlers, validators, policies, entidades, builders, mappers, readers, extensions, triggers ou procedures. Não criar Policy/Validator para retirar a regra da Service; trocar o nome da classe não atende ao requisito. Isso vale mesmo para regras puras que usam somente a biblioteca base.

Fronteiras verificam apenas aspectos técnicos: JSON legível, tipo/formato de campo, tipo de conteúdo e limite de transporte. Adaptadores de mídia medem bytes, formato e duração efetivos; a Service compara os fatos com as regras do produto. Banco protege integridade estrutural e atomicidade, como PK/FK, nulabilidade, unicidade e comparação de versão, sem assumir a decisão do caso de uso. Essa proteção não substitui a validação na Service.

### Repositórios com responsabilidade única

Repositório **somente consulta ou persiste dados**: SQL/EF parametrizado, projeção, tracking, lock, atualização condicional e tradução técnica de erro de persistência. Recebe escopo, filtros e valores explícitos, retorna dados, ausência, resultado técnico ou quantidade de linhas afetadas. Não lança EventNotFoundException porque uma consulta retornou null nem decide que uma versão divergente significa conflito de negócio; a Service interpreta esses resultados.

| Ação | Responsável |
| --- | --- |
| Consultar evento no escopo fornecido e adquirir lock solicitado. | Repository. |
| Decidir se o ator pode alterar, se o evento existe para esse ator e se a versão/prazo permite a alteração. | Service. |
| Executar `UPDATE ... WHERE id = ... AND version = ...`, retornando o resultado da escrita. | Repository. |
| Interpretar zero linhas/resultado de concorrência e lançar a exception específica. | Service. |
| Decidir alteração de acesso, cancelamento de pendências, registro de auditoria e mensagens. | Service. |
| Persistir os valores, cancelamentos e registros explicitamente solicitados. | Repository. |
| Iniciar, confirmar ou reverter a transação técnica. | UnitOfWork, sob coordenação da Service. |

O padrão `MutateAsync(..., Func<EventEntity, EventChangeModel> apply, ...)` do exemplo revisado é **proibido**: o repositório não pode executar callback de negócio, comparar versões para lançar erro funcional, decidir no-op por Changed, reagir a AccessChanged ou criar auditoria/cancelamentos implicitamente. O fluxo deve estar explícito na Service: ler com a proteção necessária, validar existência/vínculo/versão/travas, calcular a alteração, decidir seus efeitos e solicitar as operações de persistência dentro da mesma unidade de trabalho.

Não remover locks, predicados de escopo ou controles de concorrência para tornar o repositório menor. A Service define a autorização e o escopo; o repositório aplica esses filtros no SQL antes de materializar dados. Um lock solicitado continua sendo operação técnica, sem incorporar a regra que motivou sua aquisição.

Mapear explicitamente campos sensíveis, horários e enums. Biblioteca de mapper só quando houver benefício concreto, sem esconder conversão de fuso ou disponibilizar dados bloqueados.

### Convenções C#

- Uma classe/record por arquivo, com filename igual ao tipo, sempre com sufixo do conceito: EventController, EventService, EventRepository, EventEntity, CreateEventCommand, EventDetailsModel, CreateEventRequest, EventResponse, EventResponseMapper, EventRequestValidator, GlobalExceptionHandler, EventEntityConfiguration, DevelopmentIdentityOptions, BusinessClock, JsonBodyReader, HttpJsonBodyReader, HealthHttpResponseWriter, EventMvcJsonInputFormatter, EventOpenApiDocumentTransformer e EventCursorDataProtector. Validator limita-se à validação técnica de entrada.
- Extensions, Exception, Handler, Worker e Enum identificam extensões, exceções, handlers, adaptadores hospedados e enums. Interfaces usam I e mantêm o conceito (IEventService/IEventRepository/IUnitOfWork); unitários usam Tests e doubles usam Stub. Program.cs é o ponto de composição convencional.

- Nullable reference types habilitados; ausência tratada sem espalhar `!`.
- Quatro espaços; PascalCase em tipos/membros públicos, camelCase em parâmetros/locais e `Async` nos métodos assíncronos.
- Resultados/DTOs preferencialmente imutáveis; a Service valida a mudança antes de solicitar persistência.
- Composição em vez de classes base de serviço/controller com dependências acumuladas.
- Falhas de negócio e técnicas usam exceptions próprias com significado específico; tratamento HTTP centralizado e sanitizado na API.
- Recursos criados pelo código descartados; cancelamento e falhas mantêm significado.

### Código sem comentários

**É proibido adicionar comentários no código backend**, incluindo `//`, `/* ... */`, documentação XML `///`, TODO/FIXME e código morto comentado. A regra também se aplica aos exemplos de código deste guideline. Métodos devem ser pequenos, coesos e claros pelo nome, parâmetros e fluxo; não há uma quantidade arbitrária de linhas que substitua essa avaliação.

Se um trecho precisa de comentário para explicar seu funcionamento, refatorar: nomear a intenção, extrair métodos coesos, reduzir ramificações e separar responsabilidades. Regras continuam nas Services após a extração. Motivos de arquitetura, limitações externas e decisões ficam no guideline, FDD, ADR ou descrição do PR, com nomes que permitam localizar o código. Mensagens de erro seguras e contratos de dados continuam sendo parte do programa.

## 3. Injeção de dependência e execução concorrente

| Lifetime | Uso | Cuidado |
| --- | --- | --- |
| Scoped | Caso de uso, repositório e contexto de uma requisição ou mensagem. | Nunca capturar em singleton. |
| Transient | Objeto leve sem estado compartilhado. | Não presumir identidade única por request. |
| Singleton | Cliente/configuração segura para concorrência e serviço hospedado. | Sem identidade do usuário ou contexto EF retidos. |

Usar construtor e deixar o container descartar objetos que criou. Não chamar `BuildServiceProvider` no registro nem usar Service Locator no negócio. A composição na API conecta interfaces de Domain às Services de Application e aos adaptadores de Infrastructure/API, sem criar referência entre Application e Infrastructure.

Consumidores hospedados e publicador criam escopo por unidade de trabalho. Não usar um `DbContext` simultaneamente em `Task.WhenAll`; mensagens concorrentes precisam de contextos distintos. Definir limite por tarefa considerando o total de réplicas e o pool de conexões.

Encerramento deve parar novas buscas, concluir ou deixar trabalho retomável e respeitar o prazo da tarefa ECS. Não confirmar mensagem cancelada antes de persistir seu efeito.

### Falhas de consumidores e tarefas hospedadas

GlobalExceptionHandler trata o pipeline HTTP. Cada consumidor/publicador precisa de uma fronteira técnica própria de execução, com escopo por mensagem e chamada à Service de Application. O adaptador hospedado fica na API, com sufixo Worker; suas portas ficam em Domain e o acesso a filas/banco fica nos adaptadores técnicos. BackgroundService/IHostedService não entram em Application ou Domain. O Worker apenas adapta a execução e chama a Service; toda decisão de negócio permanece nas Services de Application, sem colocar uma referência a Application em Infrastructure.

| Situação | Tratamento |
| --- | --- |
| Cancelamento de encerramento | Parar novas buscas e liberar/expirar o claim; não confirmar trabalho sem conclusão durável nem classificar como falha de negócio. |
| Falha conhecida de uma mensagem | Service define o resultado de negócio; a fronteira técnica executa retry/quarentena/DLQ conforme o contrato, preserva a mensagem e registra um diagnóstico sanitizado. |
| Falha transitória de polling/transporte | Retry técnico limitado, com backoff e cancelamento; tornar a indisponibilidade observável. |
| Falha inesperada que escapa do laço de execução | Registrar o diagnóstico final e encerrar o host de forma controlada; não manter tarefa aparentemente saudável com consumidor morto. |

Manter explícito o comportamento StopHost para exceptions não tratadas de BackgroundService. Não configurar Ignore para ocultar falha fatal. O [comportamento de exceptions em serviços hospedados do .NET](https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/6.0/hosting-exception-handling) é distinto do tratamento HTTP. Limites de tentativas, deduplicação e recuperação seguem [mensageria](mensageria.md#4-consumidores-deduplicação-e-falhas); nenhum catch genérico confirma a mensagem ou continua indefinidamente após falha desconhecida.

### Validação de configuração na inicialização

**DEVE:** usar opções tipadas e validar na inicialização a configuração obrigatória das capacidades habilitadas: presença/formato, faixas técnicas, orçamento/concorrência, região/alvo, referências de segredos e combinações de flags/ambiente. Registrar a validação no host da API ou nas extensões técnicas de Infrastructure, sem referência entre Application e Infrastructure. A validação de Options é técnica; regras de negócio persistidas continuam nas Services.

Usar validação antecipada como ValidateOnStart, conforme o [padrão Options do .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/options#options-validation). Configuração obrigatória ausente/inválida impede iniciar o host; o diagnóstico informa nome da opção e motivo seguro, sem valor secreto, connection string ou payload. Startup não passa por GlobalExceptionHandler: registrar uma falha de inicialização sanitizada e terminar com resultado de falha.

Falha transitória de dependência não é o mesmo que configuração inválida: usar orçamento de inicialização/prontidão e recuperação limitada, sem reinício em massa por indisponibilidade comum do banco. Capacidade explicitamente desabilitada permanece indisponível conforme seu contrato; não ativar identidade local, emulador, credencial alternativa ou acesso público como fallback. Cobrir configurações ausentes, combinações inválidas e separação entre Development e ambientes publicados.

## 4. EF Core e acesso ao PostgreSQL

Mapear explicitamente schema, tabela, coluna, nulabilidade, tamanho, geração e tipos nas configurações da Infrastructure. O mapeamento acompanha o modelo físico aprovado, sem criá-lo.

**NÃO usar:** migrations EF, `EnsureCreated`, `Database.Migrate` ou seed estrutural no startup. Liquibase é o único responsável pelo schema.

Consultas devem ter:

- Escopo e condições definidos pela Service, aplicados por parâmetros antes da materialização; nunca materializar conteúdo sem autorização para depois descartá-lo.
- Projeção mínima e `AsNoTracking` quando apropriado.
- Limite e ordem determinística; galeria por cursor, até 10 itens.
- Filtros no banco, sem baixar tudo para filtrar no navegador.
- SQL parametrizado e allowlist para ordenações dinâmicas.
- Análise de N+1, planos e índices com volume representativo.

Não expor `IQueryable` pela porta do caso de uso. Não executar contagem para tela bloqueada nem buscar conteúdo apenas para descartá-lo depois no mapper.

Escritas relacionadas da operação compartilham unidade de trabalho coordenada pela Service. Delimitar transação explícita quando envolver leituras protegidas, múltiplas etapas ou concorrência. Manter curta e não aguardar S3/Cognito/SES com locks abertos. Repository não confirma sozinho uma transação composta nem dispara efeitos adicionais ao salvar uma entidade.

Constraints de integridade e mecanismos atômicos de concorrência são salvaguardas técnicas. Não transferir validações/transições de negócio para SQL, triggers ou procedures. Scripts e decisões anteriores permanecem evidência histórica; adequar implementações existentes em refatoração própria, com validação de concorrência e sem reescrever migrations já aplicadas.

## 5. Concorrência, idempotência e estados

As Services decidem todas as condições e transições abaixo; repositórios e UnitOfWork executam apenas as operações técnicas solicitadas, preservando atomicidade.

| Fluxo | Proteção necessária |
| --- | --- |
| Aceite de convite | Expiração, uso único, substituição, até dois responsáveis e estado de travamento revalidados atomicamente. |
| Alteração do evento | Impedir perda de atualização e mudança de campo já travado. |
| Abertura | Aplicar todas as transições e a única mudança pós-janela permitida pelo PRD. |
| Upload | Uma tentativa lógica por chave/escopo; confirmação integral do conjunto, sem novo registro em replay. |
| Primeiro registro | Evento persistido na Outbox e consumidor idempotente de travamento, com janela eventual aceita. |
| Processamento | Estado e referência externa persistentes; evento atrasado não regride trabalho concluído. |
| Sessão | Revogação/expiração consistentes entre tarefas e reconciliação com Cognito. |

Locks, constraints e atualização condicional serão definidos no modelo físico/FDD. Se usar versão otimista, configurar comparação real e tratar conflito; incrementar contador sem compará-lo não impede sobrescrita.

Persistir resultado idempotente seguro junto da operação local, com unicidade sob concorrência. Fingerprint deve detectar chave reutilizada com conteúdo diferente. Nunca guardar cookie, senha ou URL assinada para reconstruir resposta.

Recebimento confirmado é fato persistente, separado do estado de conversão. Uma falha MediaConvert não apaga o registro nem seus originais. Banco e S3 exigem reconciliação, conforme [Mensageria e mídias](mensageria.md).

## 6. Datas, prazos e identificadores

- PKs relacionais são inteiras `id bigint GENERATED ALWAYS AS IDENTITY`, mapeadas para `long`; FKs seguem tipo e [nomenclatura SQL](sql.md). Identificadores técnicos e externos têm campos próprios, com UUID quando sua semântica o exigir.
- Quando um ID bigint for exposto em JSON, usar string decimal e tratá-lo como opaco no cliente. Não inferir autorização de uma sequência numérica.
- Datas/horários de negócio usam `timestamp without time zone`, interpretados em `America/Sao_Paulo`.
- Não gravar UTC nesse tipo e depois ler como Brasília. Normalizar timestamps externos explicitamente.
- Relógios, offsets, tipos .NET/provider e serialização precisam de mapeamento testado.
- Usar relógio substituível para testar abertura, meio-dia, 13h, meia-noite, expiração de convite e sessão.
- Obter referência temporal coerente por decisão/transação; revalidar quando houver espera capaz de atravessar prazo.
- Contagem regressiva é apresentação; autorização usa relógio do servidor.
- A Service aplica a regra de datas futuras e sugestão de um ano, respeitando 29 de fevereiro → 28 de fevereiro no ano seguinte.

A conversão para horário de negócio deve usar regra de fuso explícita e testável, sem depender do timezone do host. Telemetria técnica usa instantes UTC e duração monotônica.

## 7. Async, cancelamento e resiliência

**DEVE:** manter execução assíncrona de ponta a ponta para I/O. Métodos assíncronos terminam em `Async` e retornam `Task`/`Task<T>`; `ValueTask` exige necessidade demonstrada. Não usar `.Result`, `.Wait()`, `.GetAwaiter().GetResult()`, `async void`, fire-and-forget ou `Task.Run` para esconder I/O bloqueante. Utilitários puramente em memória não precisam de async/token artificiais.

`CancellationToken` é o último parâmetro obrigatório nas entradas e portas canceláveis. Propagar o token recebido pela cadeia controller → Service → Repository/adaptador, incluindo consultas EF, SaveChangesAsync, transações, HTTP/SDKs, streams e esperas que aceitem cancelamento. Na requisição, a origem é RequestAborted; em processamento hospedado, usar o token de encerramento e o orçamento da operação. Não substituir por None/default nem criar um token desconectado para ignorar cancelamento.

Cancelamento no .NET é cooperativo. Quando houver orçamento próprio, vinculá-lo ao token de origem e descartar o CancellationTokenSource criado. Loops longos devem observar cancelamento nos pontos adequados. A [documentação de cancelamento do .NET](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads) descreve propagação e vínculo de tokens.

Não capturar OperationCanceledException para devolver sucesso, null, coleção vazia ou exception de negócio. Cancelamento esperado do cliente não vira 500 nem tentativa automática; se a conexão foi encerrada, não tentar escrever Problem Details. Distinguir esse caso de timeout interno e falha de dependência; a API mantém o contrato HTTP documentado para cada situação.

Processamento pesado respeita o modelo aprovado: fotos com concorrência limitada; vídeos/áudios no MediaConvert. Não usar um DbContext simultaneamente em Task.WhenAll. Não iniciar trabalho fora do escopo para sobreviver à requisição; persistir a intenção via Outbox quando o fluxo exigir continuidade.

Retry técnico requer causa transitória, limite, backoff, cancelamento e segurança de repetição. A Service decide tentativas/compensações de negócio; adaptadores apenas executam a política técnica explicitamente configurada. Não repetir cegamente escritas, operação Cognito ou criação de job externo.

Cancelamento ou resposta perdida depois do commit não desfaz a operação. HTTP usa idempotência/consulta de estado; consumidor usa persistência/deduplicação. Tentativas esgotadas devem ser recuperáveis por procedimento, sem loop infinito.

## 8. Autenticação, autorização e conteúdo

Seguir Cognito gerenciado, fluxo iniciado/concluído pelo backend e sessão local PostgreSQL. Cookie contém apenas identificador aleatório, com `HttpOnly`, `Secure` e política `SameSite` definida junto dos domínios e callback.

Validar OIDC, origem e CSRF. Criptografar tokens persistidos; compartilhar e proteger chaves entre réplicas, mantendo leitura de sessões válidas após reinício e rotação. Não colocar tokens Cognito em JSON para JavaScript.

Cadastro e novo e-mail exigem confirmação. Recuperação usa código Cognito de uma hora. Troca/recuperação de senha revoga todas as sessões; detalhar falha parcial e reconciliação antes de publicar o fluxo.

Autorizar cada pedido por conta/evento e abertura. Antes da data, não emitir URLs, miniaturas, contagens ou remetentes. Após mudar para bloqueio, impedir novas autorizações; URLs já emitidas têm tolerância de até 2 minutos, conforme HLD.

Separar permissões de capas públicas via CloudFront das lembranças privadas. Não servir originais como escape de falha de processamento. Credenciais habituais de operação não devem permitir leitura direta do conteúdo.

## 9. HTTP e operação

Aplicar [API](api.md) para `/api/v1`, cursor, status, erros, headers e cache. Documentar redirecionamentos e callbacks OIDC separadamente das respostas JSON.

### Exceptions próprias e GlobalExceptionHandler

**DEVE:** definir uma exception própria para cada necessidade de falha com significado distinto. Exemplos: EventNotFoundException, EventVersionConflictException, EventStructureLockedException, SubmissionWindowClosedException, RequestValidationException e PersistenceUnavailableException. Não representar todas as necessidades por `Exception`, `InvalidOperationException` ou uma única EventApplicationException com enum genérico. Uma base comum é permitida, mantendo os tipos específicos. Ausência normal de uma consulta não é exception; a Service decide quando ela representa falha do caso de uso.

As exceptions compartilhadas ficam em Domain/Exceptions, com dados seguros, sem HttpStatusCode, ProblemDetails, DbUpdateException, Npgsql ou qualquer dependência de framework. A Service lança as exceptions de negócio; fronteiras lançam as técnicas correspondentes. Infrastructure traduz somente falhas conhecidas do provider para tipos técnicos, preservando InnerException. Erros inesperados seguem ao handler como falha interna; não inventar uma classificação de negócio a partir do texto do provider.

**A API deve ter `ExceptionHandlers/GlobalExceptionHandler.cs`, implementando `IExceptionHandler`.** Ele concentra o mapeamento exception → status/código/Problem Details, sanitização e correlação dos logs. Não colocar try/catch repetido em controllers/Services para montar HTTP, nem deixar Infrastructure/Application produzir respostas HTTP. Serviços podem capturar uma falha para rollback, recuperação ou tradução semântica necessária, preservando causa e cancelamento; não ocultar a falha.

O registro usa AddExceptionHandler, AddProblemDetails e o middleware UseExceptionHandler antes dos componentes cujas exceptions deve tratar, conforme a [documentação de tratamento de erros do ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-10.0):

```csharp
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
```

Depois de construir o app, registrar o middleware antes dos endpoints. O handler é singleton; não capturar Services, repositórios, contexto EF ou outros objetos scoped no construtor:

```csharp
app.UseExceptionHandler();
app.MapControllers();
```

O handler não aplica regras de negócio, consulta repositórios ou tenta novamente a operação. Não expor SQL, stack trace, credenciais, payload privado nem mensagem bruta de InnerException. Não reescrever resposta já iniciada; preservar o tratamento de cancelamento descrito na seção 7. Definir o registro de telemetria em um único ponto para evitar duplicidade.

| Necessidade | Tipo ilustrativo | Tradução na API |
| --- | --- | --- |
| Evento inexistente ou oculto por decisão da Service. | EventNotFoundException | 404 / RESOURCE_NOT_FOUND. |
| Versão não permite a alteração. | EventVersionConflictException | 409 / EVENT_VERSION_CONFLICT. |
| Estrutura travada. | EventStructureLockedException | 409 / EVENT_LOCKED. |
| Janela para novo envio encerrada. | SubmissionWindowClosedException | 409 / UPLOAD_WINDOW_CLOSED. |
| Entrada inválida. | RequestValidationException | 400 / REQUEST_VALIDATION_FAILED, com campos seguros. |
| Dependência reconhecidamente indisponível. | PersistenceUnavailableException | 503 / DEPENDENCY_UNAVAILABLE. |
| Falha inesperada. | Exception não reconhecida | 500 / UNEXPECTED_ERROR, sanitizado. |

Essa tabela exemplifica o padrão; cada fluxo deve registrar os tipos e códigos necessários no [catálogo HTTP](api.md#4-erros-e-catálogo-inicial). MVC, autenticação e rejeições técnicas que produzem erro sem lançar exception devem usar a mesma fábrica/formato de Problem Details da API; não criar exceptions artificiais só para alterar o status.

### Operação e implantação

Health deve distinguir vitalidade do processo e prontidão para o ALB. Não expor configuração ou conteúdo. Falha de banco não deve causar reinício em massa sem análise. Endpoints operacionais e OpenAPI publicado precisam de acesso compatível com o ambiente e política explícita.

A base disponibiliza `GET /health` fora de `/api/v1`, com JSON `{"status":"up"}` e HTTP 200; resultados degraded/unhealthy têm mapeamento 503/`down`. Sem verificações externas registradas, o endpoint comprova apenas que a API responde. O frontend consulta esse recurso por ação explícita usando o proxy local.

A implantação segue a topologia aprovada: `us-east-1`, VPCs separadas para homologação/produção, ALB público com WAF e tarefas com entrada restrita ao security group do ALB. RDS permanece privado; produção usa Multi-AZ e ao menos duas tarefas Fargate em duas zonas. A configuração inicial usa Gateway Endpoint S3 e não inclui NAT Gateway.

Documentar backups automáticos RDS de 35 dias, recuperação para ponto no tempo, versionamento S3 e KMS separado por ambiente. Validar recuperação conjunta de metadados e arquivos. Calibrar WAF/login para o Wi-Fi compartilhado; controles adicionais por conta ou sessão de upload exigem decisão própria, conforme o escopo do HLD.

Segredos locais em configuração privada ignorada; ambientes AWS via Secrets Manager e IAM de tarefa, sem chaves permanentes. Identidade de migração DDL distinta da aplicação DML.

Logs estruturados, correlação e instrumentos seguem [Observabilidade](observabilidade.md). Não habilitar logging de dados sensíveis do EF. Confirmar nomes de métricas e comportamento do handler na versão escolhida, sem transportar regras específicas do .NET 10 como fato já verificado.

### Containers locais

API possui Dockerfile multi-stage com SDK 10.0.401 e runtime ASP.NET 10.0.12/noble fixados por digest. Restaurar com --locked-mode e publicar Release; runtime sem SDK, usuário app, porta interna 8080 e healthcheck HTTP. Preservar timezone/ICU necessários ao BusinessClock. Contexto de build exclui bibliotecas locais, chaves e configuração privada.

Compose em scrapbook-infrastructure conecta service a postgres:5432 com credencial DML própria; nunca usar administrador/migrador no backend. API/UI têm filesystem somente leitura e /tmp temporário. Chaves de cursor usam volume persistente independente do banco. Não migrar ou criar fixture no startup. Identidade local em 0.0.0.0 exige opt-in AllowContainerBinding, indicação de container e Development; manter desligada por padrão, com publicação somente em loopback. Produção continua recusando a simulação.

Construção/operação local em feature/6: [infraestrutura](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/feature/6/README.md). Validação manual da stack em scrapbook-tests/docker_stack.py, com rede/volumes descartáveis próprios. Não confundir essa entrega com deploy ECS/AWS.

## 10. Testes e revisão

Somente unitários no backend, com xUnit 2.9.3. Executar em `scrapbook-service`:

```bash
dotnet restore ScrapBook.sln --locked-mode
dotnet build ScrapBook.sln --no-restore
dotnet test ScrapBook.sln --no-build --no-restore
```

Testar decisões de negócio nas Services com portas substituídas; verificar os tipos específicos das exceptions, efeitos solicitados e propagação de cancelamento. Testes de arquitetura devem verificar a matriz exata de ProjectReference e a localização dos contratos, sem declarar conformidade apenas porque a classe se chama Service. Unitários do GlobalExceptionHandler verificam mapeamento, sanitização e cancelamento. Testes HTTP com host de teste, PostgreSQL real, serviços AWS e falhas de integração ficam no repositório de testes, acionados manualmente; `WebApplicationFactory` permanece uma opção para essa suíte.

Não usar EF InMemory ou SQLite como comprovação de SQL, locks, constraints ou datas do PostgreSQL. Simulações locais aceleram desenvolvimento, mas não comprovam integração real com Cognito/MediaConvert/SQS.

- [ ] Toda regra de negócio está em Service de Application; Domain e demais classes não decidem regras, invariantes ou transições.
- [ ] Repositories fazem somente persistência, sem callbacks de negócio, decisões funcionais, auditoria ou cancelamentos implícitos.
- [ ] ProjectReference segue exatamente API → Application/Infrastructure/Domain; Application → Domain; Infrastructure → Domain; Domain → nenhum projeto.
- [ ] Interfaces, contratos compartilhados e exceptions estão em Domain; Application/Infrastructure não importam tipos uma da outra nem dependem de frameworks indevidos.
- [ ] Diretórios e sufixos representam conceitos, sem Modules/Endpoints ou subpastas funcionais.
- [ ] Métodos são pequenos e claros, com responsabilidade única, sem comentários, XML de documentação, TODO/FIXME ou código comentado.
- [ ] Falhas distintas possuem exceptions próprias e mapeamento no GlobalExceptionHandler da API, com mensagens seguras e sem catches HTTP espalhados.
- [ ] Async/await e CancellationToken são propagados por todas as entradas/portas e operações de I/O canceláveis, sem bloqueio, descarte do token ou cancelamento convertido em sucesso/500.
- [ ] Lifetimes e concorrência protegem contexto/pool/API; não há operações simultâneas no mesmo DbContext.
- [ ] Workers tratam falhas por mensagem, cancelamento e falha fatal do host, sem depender do handler HTTP ou esconder consumidor morto.
- [ ] Opções obrigatórias são validadas no startup; configuração inválida falha com diagnóstico seguro e sem fallback permissivo.
- [ ] Vínculo, abertura e prazo são verificados no servidor.
- [ ] Confirmação, transação local, Outbox e idempotência estão alinhadas.
- [ ] Sessão, chaves e revogação funcionam entre réplicas.
- [ ] Mapeamento corresponde ao Liquibase e preserva datas de Brasília.
- [ ] Testes cobrem sucesso, limite, falha parcial, repetição e disputa pertinentes.
- [ ] Build/unitários e execuções manuais necessárias têm evidência, sem testes ignorados tratados como aprovação.

### Adoção no código existente

Esta revisão é documental. O backend existente ainda contém portas/exceptions em Application, referência Infrastructure → Application, regras em Domain/Policies e EventRepository.MutateAsync, e tradução de erros em EventRequestMiddleware. A API também precisa declarar referência direta a Domain. Esses pontos não atendem à estrutura-alvo e devem ser refatorados em entrega própria, com regressão de autorização, privacidade, transações e concorrência. Não usar testes ou relatórios antigos como comprovação de atendimento às regras de 2026-11-15.
