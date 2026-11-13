# Guideline de backend

[Índice](README.md) · [Princípios](principios.md) · [Contratos HTTP](api.md) · [Mensageria e mídias](mensageria.md) · [Observabilidade](observabilidade.md)

Aplicação: C#/.NET e ASP.NET Core 10. SDK 10.0.401, runtime validado 10.0.12, solution `ScrapBook.sln` e xUnit 2.9.3. A feature Gestão de Eventos fixa EF Core 10.0.15, Npgsql/provider 10.0.3 e Microsoft.AspNetCore.OpenApi 10.0.15, com lockfiles. Consulte o [backend na feature](https://github.com/diorgenesbackes/scrapbook-service/blob/feature/GE-STORY-001/README.md). Integrações AWS permanecem pendentes; essa branch não representa publicação.

## 1. Clean Architecture por conceito

Decisão do responsável em **2026-11-13**: serviço único organizado por camadas e conceitos, registrado no [ADR-001](../adr/ADR-001-backend-clean-architecture.md). Retirar Modules, Endpoints e subpastas funcionais. A API utiliza controllers MVC reais, derivados de ControllerBase, com ApiController e rotas por atributos; composição por AddControllers/MapControllers.

| Responsabilidade | Conteúdo | Dependências permitidas |
| --- | --- | --- |
| Domain | Entidades, valores, invariantes e transições. | Biblioteca base; sem HTTP, EF Core ou SDK AWS. |
| Application | Casos de uso, autorização, coordenação, portas e resultados. | Domain e contratos internos explicitamente definidos. |
| Infrastructure | EF Core e adaptadores Cognito/S3/SQS/SNS/SES/MediaConvert. | Domain/Application cujas portas implementa. |
| API / host | DTOs HTTP, endpoints, autenticação, erros, composição e hospedagem dos consumidores. | Application; Infrastructure no ponto de composição. |
| UnitTests | Testes unitários com dependências substituídas. | Unidades sob teste. |
| Repositório de testes | Integração, HTTP, banco, navegador, carga, segurança e recuperação. | Componentes necessários ao cenário, por acionamento manual. |

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
| API | Controllers, Requests, Responses, Middlewares, Validators, Mappers, Options, Services, Readers, Writers, InputFormatters, Transformers, Models, Extensions |
| Application | Interfaces, Services, Commands, Models, Builders, Exceptions, Enums, Scopes |
| Domain | Entities, Policies, Models, Exceptions, Enums, Clocks |
| Infrastructure | Repositories, DbContexts, Configurations, Entities, Services, Mappers, UnitOfWorks, Extensions |
| UnitTests | Conceito testado, com doubles em Stubs e providers em Providers |

EventController depende de IEventService; EventService utiliza IEventRepository e IUnitOfWork. Infraestrutura implementa repositórios e um ScrapBookDbContext scoped, compartilhado na requisição, com IEntityTypeConfiguration separado por entidade. O modelo atual mapeia cinco entidades usadas pelas funcionalidades existentes. Os schemas físicos identity/events/records/notifications e as chaves existentes de configuração/telemetria não definem módulos de código e continuam compatíveis.

Operação local e Outbox, quando aplicável, compartilham transação curta. Integrações externas não participam de locks/transações do banco. Liquibase permanece o executor exclusivo de DDL; startup não migra ou prepara dados.

DTOs HTTP pertencem à API; entradas/resultados independentes de transporte, a Application. Não serializar entidade EF como resposta nem colocar contrato HTTP em Domain.

## 2. Fluxo de um caso de uso

O endpoint recebe entrada, identidade/autorização aplicável e cancelamento; chama Application e traduz o resultado. Application autoriza, consulta portas, aplica regras, coordena transação e retorna resultado. Domain impede estados inválidos. Infrastructure executa I/O.

Validação é complementar:

- Fronteira: formato, tamanho, quantidade, duração declarada e campos permitidos.
- Caso de uso: vínculo, janela, abertura, idempotência e pré-condições.
- Domínio: invariantes e transições.
- Banco: integridade, unicidade e concorrência persistente.
- Adaptador de mídia: tamanho, duração e conteúdo efetivos.

Mapear explicitamente campos sensíveis, horários e enums. Biblioteca de mapper só quando houver benefício concreto, sem esconder conversão de fuso ou disponibilizar dados bloqueados.

### Convenções C#

- Uma classe/record por arquivo, com filename igual ao tipo, sempre com sufixo do conceito: EventController, EventService, EventRepository, EventEntity, CreateEventCommand, EventDetailsModel, CreateEventRequest, EventResponse, EventResponseMapper, EventRequestValidator, EventRequestMiddleware, EventEntityConfiguration, DevelopmentIdentityOptions, EventPolicy, BusinessClock, JsonBodyReader, HealthResponseWriter, EventJsonInputFormatter e EventOpenApiDocumentTransformer.
- Extensions, Exception e Enum identificam extensões, exceções e enums. Interfaces usam I e mantêm o conceito (IEventService/IEventRepository/IUnitOfWork); unitários usam Tests e doubles usam Stub. Program.cs é o ponto de composição convencional.

- Nullable reference types habilitados; ausência tratada sem espalhar `!`.
- Quatro espaços; PascalCase em tipos/membros públicos, camelCase em parâmetros/locais e `Async` nos métodos assíncronos.
- Resultados/DTOs preferencialmente imutáveis; mutações de entidades por operações que preservem invariantes.
- Composição em vez de classes base de serviço/controller com dependências acumuladas.
- Resultado esperado de domínio distinto de exceção técnica; tratamento centralizado e sanitizado.
- Recursos criados pelo código descartados; cancelamento e falhas mantêm significado.

## 3. Injeção de dependência e execução concorrente

| Lifetime | Uso | Cuidado |
| --- | --- | --- |
| Scoped | Caso de uso, repositório e contexto de uma requisição ou mensagem. | Nunca capturar em singleton. |
| Transient | Objeto leve sem estado compartilhado. | Não presumir identidade única por request. |
| Singleton | Cliente/configuração segura para concorrência e serviço hospedado. | Sem identidade do usuário ou contexto EF retidos. |

Usar construtor e deixar o container descartar objetos que criou. Não chamar `BuildServiceProvider` no registro nem usar Service Locator no negócio.

Consumidores hospedados e publicador criam escopo por unidade de trabalho. Não usar um `DbContext` simultaneamente em `Task.WhenAll`; mensagens concorrentes precisam de contextos distintos. Definir limite por tarefa considerando o total de réplicas e o pool de conexões.

Encerramento deve parar novas buscas, concluir ou deixar trabalho retomável e respeitar o prazo da tarefa ECS. Não confirmar mensagem cancelada antes de persistir seu efeito.

## 4. EF Core e acesso ao PostgreSQL

Mapear explicitamente schema, tabela, coluna, nulabilidade, tamanho, geração e tipos nas configurações da Infrastructure. O mapeamento acompanha o modelo físico aprovado, sem criá-lo.

**NÃO usar:** migrations EF, `EnsureCreated`, `Database.Migrate` ou seed estrutural no startup. Liquibase é o único responsável pelo schema.

Consultas devem ter:

- Escopo autorizado por evento e condição temporal antes da materialização.
- Projeção mínima e `AsNoTracking` quando apropriado.
- Limite e ordem determinística; galeria por cursor, até 10 itens.
- Filtros no banco, sem baixar tudo para filtrar no navegador.
- SQL parametrizado e allowlist para ordenações dinâmicas.
- Análise de N+1, planos e índices com volume representativo.

Não expor `IQueryable` pela porta do caso de uso. Não executar contagem para tela bloqueada nem buscar conteúdo apenas para descartá-lo depois no mapper.

Escritas relacionadas da operação compartilham unidade de trabalho. Delimitar transação explícita quando envolver leituras protegidas, múltiplas etapas ou concorrência. Manter curta e não aguardar S3/Cognito/SES com locks abertos.

## 5. Concorrência, idempotência e estados

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
- Datas futuras e sugestão de um ano respeitam 29 de fevereiro → 28 de fevereiro no ano seguinte.

A conversão para horário de negócio deve usar regra de fuso explícita e testável, sem depender do timezone do host. Telemetria técnica usa instantes UTC e duração monotônica.

## 7. Async, cancelamento e resiliência

Propagar `CancellationToken` pelas operações de I/O. Não usar `.Result`, `.Wait()`, `async void` ou `Task.Run` para esconder I/O bloqueante. Processamento pesado deve respeitar o modelo aprovado: fotos com concorrência limitada; vídeos/áudios no MediaConvert.

Definir orçamento por operação e distinguir cliente cancelado, timeout interno e falha externa. Retry requer causa transitória, limite, backoff e segurança de repetição. Não repetir cegamente escritas, operação Cognito ou criação de job externo.

Cancelamento ou resposta perdida depois do commit não desfaz a operação. HTTP usa idempotência/consulta de estado; consumidor usa persistência/deduplicação. Tentativas esgotadas devem ser recuperáveis por procedimento, sem loop infinito.

## 8. Autenticação, autorização e conteúdo

Seguir Cognito gerenciado, fluxo iniciado/concluído pelo backend e sessão local PostgreSQL. Cookie contém apenas identificador aleatório, com `HttpOnly`, `Secure` e política `SameSite` definida junto dos domínios e callback.

Validar OIDC, origem e CSRF. Criptografar tokens persistidos; compartilhar e proteger chaves entre réplicas, mantendo leitura de sessões válidas após reinício e rotação. Não colocar tokens Cognito em JSON para JavaScript.

Cadastro e novo e-mail exigem confirmação. Recuperação usa código Cognito de uma hora. Troca/recuperação de senha revoga todas as sessões; detalhar falha parcial e reconciliação antes de publicar o fluxo.

Autorizar cada pedido por conta/evento e abertura. Antes da data, não emitir URLs, miniaturas, contagens ou remetentes. Após mudar para bloqueio, impedir novas autorizações; URLs já emitidas têm tolerância de até 2 minutos, conforme HLD.

Separar permissões de capas públicas via CloudFront das lembranças privadas. Não servir originais como escape de falha de processamento. Credenciais habituais de operação não devem permitir leitura direta do conteúdo.

## 9. HTTP e operação

Aplicar [API](api.md) para `/api/v1`, cursor, status, erros, headers e cache. Documentar redirecionamentos e callbacks OIDC separadamente das respostas JSON.

Health deve distinguir vitalidade do processo e prontidão para o ALB. Não expor configuração ou conteúdo. Falha de banco não deve causar reinício em massa sem análise. Endpoints operacionais e OpenAPI publicado precisam de acesso compatível com o ambiente e política explícita.

A base disponibiliza `GET /health` fora de `/api/v1`, com JSON `{"status":"up"}` e HTTP 200; resultados degraded/unhealthy têm mapeamento 503/`down`. Sem verificações externas registradas, o endpoint comprova apenas que a API responde. O frontend consulta esse recurso por ação explícita usando o proxy local.

A implantação segue a topologia aprovada: `us-east-1`, VPCs separadas para homologação/produção, ALB público com WAF e tarefas com entrada restrita ao security group do ALB. RDS permanece privado; produção usa Multi-AZ e ao menos duas tarefas Fargate em duas zonas. A configuração inicial usa Gateway Endpoint S3 e não inclui NAT Gateway.

Documentar backups automáticos RDS de 35 dias, recuperação para ponto no tempo, versionamento S3 e KMS separado por ambiente. Validar recuperação conjunta de metadados e arquivos. Calibrar WAF/login para o Wi-Fi compartilhado; controles adicionais por conta ou sessão de upload exigem decisão própria, conforme o escopo do HLD.

Segredos locais em configuração privada ignorada; ambientes AWS via Secrets Manager e IAM de tarefa, sem chaves permanentes. Identidade de migração DDL distinta da aplicação DML.

Logs estruturados, correlação e instrumentos seguem [Observabilidade](observabilidade.md). Não habilitar logging de dados sensíveis do EF. Confirmar nomes de métricas e comportamento do handler na versão escolhida, sem transportar regras específicas do .NET 10 como fato já verificado.

## 10. Testes e revisão

Somente unitários no backend, com xUnit 2.9.3. Executar `dotnet build ScrapBook.sln` e `dotnet test ScrapBook.sln --no-build` em `scrapbook-service`. Os testes iniciais exercitam a resposta de saúde em memória, sem iniciar servidor HTTP, e verificam a ausência de detalhes sensíveis. Testes HTTP com host de teste, PostgreSQL real, serviços AWS e falhas de integração ficam no repositório de testes, acionados manualmente; `WebApplicationFactory` permanece uma opção para essa futura suíte.

Não usar EF InMemory ou SQLite como comprovação de SQL, locks, constraints ou datas do PostgreSQL. Simulações locais aceleram desenvolvimento, mas não comprovam integração real com Cognito/MediaConvert/SQS.

- [ ] Serviços e políticas independem de HTTP/AWS; interfaces e implementações respeitam as camadas.
- [ ] Diretórios e sufixos representam conceitos, sem Modules/Endpoints ou subpastas funcionais.
- [ ] Lifetimes, cancelamento e concorrência protegem contexto/pool/API.
- [ ] Vínculo, abertura e prazo são verificados no servidor.
- [ ] Confirmação, transação local, Outbox e idempotência estão alinhadas.
- [ ] Sessão, chaves e revogação funcionam entre réplicas.
- [ ] Mapeamento corresponde ao Liquibase e preserva datas de Brasília.
- [ ] Testes cobrem sucesso, limite, falha parcial, repetição e disputa pertinentes.
- [ ] Build/unitários e execuções manuais necessárias têm evidência, sem testes ignorados tratados como aprovação.
