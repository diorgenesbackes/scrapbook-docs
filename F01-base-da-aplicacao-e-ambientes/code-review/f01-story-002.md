# Code review da F01-STORY-002

**Parecer técnico em 19/11/2026. Corrigir os achados P2 antes do push.**

| Repositório | Base local `origin/dev` | Commit revisado em `feature/F01-STORY-002` |
| --- | --- | --- |
| `scrapbook-service` | `9a32dce` | `7375858` |
| `scrapbook-ui` | `1ecfe5f` | `af9ae87` |
| `scrapbook-infrastructure` | `f2b4388` | `2d51fe4` |
| `scrapbook-tests` | `175565b` | `cdda498` |
| `scrapbook-docs` | `b45b319` | `2c8b0b9` |

O Liquibase `1adeced` foi consultado e executado como dependência. A comparação de documentação usa o commit e exclui as alterações de trabalho da F01-STORY-001 e da F03. O escopo abrange implementação e contratos técnicos de inicialização, HTTP, PostgreSQL, proxy e recuperação. [Spec da F01](../spec.md) · [Guidelines](../../guidelines/README.md)

## Achados P2

### P2 · Uma preparação que falha pode interromper uma stack já funcional

O script confirma a troca das senhas das roles de aplicação e migração antes de executar o launcher do Liquibase. Em uma stack descartável já pronta, o launcher ausente fez o comando sair com código `1` após a troca de uma senha sintética. PostgreSQL registrou a alteração e, após reiniciar a API com sua configuração original, `/health` retornou `200` e `/health/ready` caiu de `200` para `503`. A mensagem final também descartou a identificação da etapa que falhou.

Validar a disponibilidade do migrador antes dos efeitos administrativos. Preservar a credencial ativa no preparo recorrente, tratar rotação como ação explícita e informar a etapa e a causa sanitizada quando o preparo falhar. Introduzido e reproduzido. [Preparo local, linhas 16–18 e 41–58](../../../scrapbook-infrastructure/scripts/prepare_database.py) · [Recuperação do Liquibase](../../guidelines/liquibase.md#8-preconditions-drift-e-recuperação)

### P2 · Uma queda do PostgreSQL escapa da classificação sanitizada de prontidão

A camada EF/Npgsql envolveu a falha de conexão em `InvalidOperationException`. A sondagem captura apenas `NpgsqlException` direta. Ao parar o banco descartável, o endpoint manteve `503` com JSON seguro, mas `DefaultHealthCheckService` registrou a exceção e a causa Npgsql com stack trace. A categoria sanitizada de indisponibilidade ficou ausente nessa execução.

Traduzir falhas conhecidas da cadeia do provider na fronteira de persistência, preservar a causa interna e emitir o motivo sanitizado no health check. Manter a distinção entre timeout, cancelamento do cliente e falha de conexão. Introduzido e reproduzido. [Sondagem, linha 43](../../../scrapbook-service/ScrapBook.Infrastructure/Probes/EfPostgresReadinessProbe.cs) · [Tratamento de exceptions](../../guidelines/backend.md#exceptions-próprias-e-globalexceptionhandler) · [Dados permitidos em logs](../../guidelines/observabilidade.md#dados-permitidos)

### P2 · O Nginx registra query strings brutas

A configuração mantém o log de acesso padrão. Uma requisição à UI com query sintética retornou `200` e seu valor apareceu em `docker compose logs ui`. A rota nominal, o status e a correlação bastam para a finalidade do log e evitam registrar parâmetros de URL.

Definir um formato de acesso sem query string e verificar que a mesma regra se mantém no diagnóstico de falha de proxy. Introduzido e reproduzido. [Configuração Nginx, linha 5](../../../scrapbook-ui/nginx/default.conf) · [Dados permitidos em logs](../../guidelines/observabilidade.md#dados-permitidos)

### P2 · A identidade de desenvolvimento aceita binding amplo sem a autorização específica

Com `Enabled=true` e credenciais válidas, a validação só exige `Development`. Ela aceita a configuração de escuta em todas as interfaces do contêiner sem verificar `AllowContainerBinding` e a indicação de que o host roda em contêiner. O loopback da porta publicada pelo Compose limita o acesso pelo host e a escuta dentro do contêiner continua ampla.

Vincular a validação da identidade à configuração efetiva de escuta. Exigir o opt-in adicional e a indicação de contêiner para binding amplo e testar as combinações aceitas e recusadas. Introduzido e identificado por inspeção. [Validação da identidade, linha 19](../../../scrapbook-service/ScrapBook.Api/Options/DevelopmentIdentityOptions.cs) · [Compose, linha 24](../../../scrapbook-infrastructure/compose.yaml) · [Containers locais](../../guidelines/backend.md#containers-locais)

### P2 · A composição React carece de recuperação para falhas de renderização

O ponto de entrada monta `App` diretamente em `StrictMode`. A rota desconhecida só trata a URL. Se uma renderização lançar uma exceção, a aplicação carece de uma fronteira para oferecer uma mensagem segura e uma ação de recuperação.

Adicionar um Error Boundary na composição da aplicação com fallback e ação explícita para recuperação. Exercitar a fronteira com um erro de renderização controlado. Introduzido e identificado por inspeção. [Composição, linha 9](../../../scrapbook-ui/src/main.tsx) · [Recuperação da interface](../../guidelines/frontend.md#recuperação-de-falhas-da-interface)

## Achados P3

| Achado | Evidência e direção de correção |
| --- | --- |
| Resumo HTTP fora do formato de observabilidade | A requisição sintética produziu um evento com `EventId=0` e sem timestamp no JSON. O middleware também omite versão, ambiente, método HTTP e classificação de resultado. Padronizar o evento terminal e seu contexto comum. [Composição, linha 31](../../../scrapbook-service/ScrapBook.Api/Program.cs) · [Middleware, linha 38](../../../scrapbook-service/ScrapBook.Api/Middlewares/CorrelationMiddleware.cs) · [Formato de logs](../../guidelines/observabilidade.md#3-formato-de-logs) |
| Transporte HTTP dentro do serviço de prontidão da UI | `checkReadiness` escolhe base URL, executa `fetch` e interpreta HTTP junto da validação do payload da operação. Centralizar o transporte e a classificação técnica de falhas em `lib` e manter a validação do payload no consumidor. `src/services/readinessService.ts`, linha 3 no commit `af9ae87`. [Integração HTTP](../../guidelines/frontend.md#6-integração-http-e-sessão) |
| `HealthResponse` está na API | O record contém somente o campo do contrato JSON e foi declarado na API. Mover o contrato de dados para Domain e manter o writer HTTP na API. `ScrapBook.Api/Responses/HealthResponse.cs`, linha 3 no commit `7375858`. [Contratos puros](../../guidelines/backend.md#contratos-puros-e-adaptadores-técnicos) |
| A inicialização da UI independe da saúde da API | O Compose inicia `ui` sem condicionar sua ordem à vitalidade de `service`. Uma inicialização isolada da UI pode anunciar Nginx saudável antes de o proxy alcançar a API. Usar uma dependência de inicialização por saúde e manter o endpoint `/ui-health` independente. [Compose, linha 53](../../../scrapbook-infrastructure/compose.yaml) · [Containers locais](../../guidelines/frontend.md#containers-locais) |
| A espera de saúde do runner pode terminar antes do prazo | Após um reinício do serviço, a sondagem de revisão chamou o mesmo helper e recebeu `http.client.RemoteDisconnected`, que escapou do loop e encerrou aquela tentativa. Tratar desconexões transitórias na espera e manter a asserção final com o último estado observado. A suíte habitual passou em sua própria execução. [Runner, linha 81](../../../scrapbook-tests/docker_stack.py) |

Os achados P3 pertencem às mudanças revisadas. A inspeção fundamenta os três desvios de estrutura e inicialização. As sondagens locais confirmaram o formato dos logs e a falha do helper de espera.

## Verificações

.NET SDK `10.0.401`, Node `26.13.0`, npm `11.20.0`, Python `3.13.15`, Docker `29.8.2` com Compose `5.6.0`, PostgreSQL `18.6` e Liquibase `4.33.0`.

| Verificação | Código de saída e resultado |
| --- | --- |
| `git diff --check origin/dev...HEAD` nos cinco repositórios | `0`, sem diagnósticos de whitespace. |
| `dotnet restore ScrapBook.sln --locked-mode --disable-build-servers` | `0`, zero avisos. |
| `dotnet build ScrapBook.sln --no-restore --no-incremental --disable-build-servers` | `0`, zero erros e zero avisos de compilação. |
| `dotnet format ScrapBook.sln --verify-no-changes --no-restore --severity info` | `2`, dois diagnósticos informativos `CA1873`. A análise sinalizou avaliação dispensável de argumentos de logging quando o nível está desativado. [CorrelationMiddleware.cs, linhas 38 e 42](../../../scrapbook-service/ScrapBook.Api/Middlewares/CorrelationMiddleware.cs) |
| `dotnet test ScrapBook.sln --no-build --no-restore --disable-build-servers` | `0`, 22 aprovados, zero falhas e zero pulados. |
| `npm run lint` e `npm run build` | `0` em cada comando, zero avisos de lint, TypeScript e Vite. |
| `npm test` | `0`, sete aprovados em dois arquivos. |
| `ast.parse` dos scripts Python da história | `0`, sintaxe válida em infraestrutura e testes. |
| `python3 docker_stack.py --browser` | `0`, um cenário Compose e dez testes Playwright aprovados. Cinco avisos do ambiente Node informaram que `NO_COLOR` foi ignorado porque `FORCE_COLOR` estava definido. Sem localização de código. |
| Sondagens técnicas isoladas | Confirmaram os achados de credencial, diagnósticos de prontidão e logs. A primeira espera após reinício direto falhou por `RemoteDisconnected`. O reteste esperou a vitalidade do contêiner e confirmou a degradação de prontidão. |

A revisão inspecionou as asserções de unitários e suites locais. As configurações atuais não trazem coletores de cobertura instalados. O AGENTS.md vigente não define percentual mínimo de cobertura. Os resultados de aceitação funcional ficam nas evidências da etapa correspondente. [Evidência funcional](../test-funcional/2026-11-19_11-07-50/2-execucao.md)

## Reteste das correções

Em 19/11/2026, a suíte descartável confirmou a correção da troca de credenciais em preparo recorrente. Com a stack pronta, executou três variações, incluindo launcher ausente, senha divergente e saída `12` do launcher sintético. O script identificou a etapa em cada falha, manteve a credencial ativa e a API voltou a responder `200` em prontidão após reiniciar com a configuração original. [Preparo local](../../../scrapbook-infrastructure/scripts/prepare_database.py) · [Suíte descartável](../../../scrapbook-tests/docker_stack.py)

Ao parar o PostgreSQL, a API manteve a vitalidade em `200`, devolveu prontidão `503` e registrou `database_unavailable` sem exceções brutas. O Nginx omitiu a query sintética nas requisições bem-sucedidas e nas falhas de proxy, mantendo rota nominal, status e correlação quando disponível. O Compose passou a aguardar a vitalidade da API antes de iniciar a UI. A parada da API manteve `/ui-health` em `200`. [Sondagem](../../../scrapbook-service/ScrapBook.Infrastructure/Probes/EfPostgresReadinessProbe.cs) · [Log do Nginx](../../../scrapbook-ui/nginx/default.conf) · [Compose](../../../scrapbook-infrastructure/compose.yaml)

O host recusou a identidade habilitada fora de `Development` e a escuta ampla sem o opt-in de contêiner. A combinação autorizada passou no Compose. Os unitários cobriram loopback IPv4 e IPv6, porta sem endereço explícito, host com nome ambíguo e configurações Kestrel. [Opções](../../../scrapbook-service/ScrapBook.Api/Options/DevelopmentIdentityOptions.cs) · [Unitários de opções](../../../scrapbook-service/ScrapBook.UnitTests/Options/StartupOptionsTests.cs)

O teste React lançou uma falha de renderização controlada, confirmou mensagem segura com foco no fallback e recuperou a tela após ação explícita. O adaptador HTTP foi centralizado em `lib` e as respostas `204` e JSON inválido passaram a ter testes. [Fronteira React](../../../scrapbook-ui/src/components/ApplicationErrorBoundary.tsx) · [Adaptador HTTP](../../../scrapbook-ui/src/lib/httpClient.ts)

A suíte conferiu evento HTTP `1000` com timestamp, versão, ambiente, rota, método, correlação e resultado. O writer usa o record de Domain. A espera de saúde após reinício absorve desconexão transitória e conserva o prazo para confirmar a prontidão. [Resumo HTTP](../../../scrapbook-service/ScrapBook.Api/Middlewares/CorrelationMiddleware.cs) · [Contrato de domínio](../../../scrapbook-service/ScrapBook.Domain/Responses/HealthResponse.cs) · [Runner](../../../scrapbook-tests/docker_stack.py)

| Verificação final | Resultado |
| --- | --- |
| Restore travado, build e unitários .NET | Código `0`, zero avisos de build e 32 testes aprovados. |
| `dotnet format ScrapBook.sln --verify-no-changes --no-restore --severity info` | Código `0`, sem diagnósticos. |
| `npm run check` | Código `0`, lint e build aprovados e 10 unitários aprovados. |
| `python3 docker_stack.py --browser` | Código `0`, cenário Compose com as regressões e dez testes Playwright aprovados. A normalização de cor eliminou os avisos de `NO_COLOR` dos workers. |

A execução cobriu as correções técnicas em ambiente local. Os ensaios manuais de leitores de tela, dispositivos físicos e matriz completa de navegadores seguem na etapa funcional.

## Segunda revisão técnica

**Parecer em 19/11/2026. Corrigir a seleção de endereços da identidade de desenvolvimento antes do push.**

| Repositório | Base local `origin/dev` | Commit revisado |
| --- | --- | --- |
| `scrapbook-service` | `9a32dce` | `8a4a4ec` |
| `scrapbook-ui` | `1ecfe5f` | `b129584` |
| `scrapbook-infrastructure` | `f2b4388` | `e676dc5` |
| `scrapbook-tests` | `175565b` | `96136d0` |
| `scrapbook-docs` | `b45b319` | `21fddbd` |

O cenário Compose usou o Liquibase `405538d` de `dev`. A revisão percorreu os commits de correção e seus consumidores. A documentação local da F01-STORY-001 e da F03 ficou fora do escopo.

### P2 · A validação da identidade pode escolher um conjunto de endereços diferente do Kestrel

`UsesOnlyLoopback` interpreta `preferHostingUrls` com `bool.TryParse`. O ASP.NET Core interpreta também `1` como verdadeiro. Quando os endpoints configurados são de loopback e os endereços do host incluem uma escuta ampla, essa diferença pode fazer a validação aceitar a identidade sem o opt-in enquanto o Kestrel prefere os endereços do host. Há uma segunda divergência da mesma lógica. A validação considera qualquer string de URLs não vazia como presença de endereço. Uma lista composta apenas por separadores fica vazia após a expansão, e o Kestrel volta aos endpoints configurados. A chamada a `All` sobre a lista vazia aceita a configuração antes de avaliá-los.

Usar a mesma semântica de leitura e precedência do runtime na validação, contando os endereços efetivos após expandir as listas. Cobrir configurações mistas nos unitários. O achado anterior fica parcialmente resolvido. As verificações padrão do Compose passaram e a divergência foi identificada por comparação com o runtime fixado, por inspeção. [Opções da API, linhas 52–70](../../../scrapbook-service/ScrapBook.Api/Options/DevelopmentIdentityOptions.cs) · [Leitura de booleanos do ASP.NET Core](https://github.com/dotnet/aspnetcore/blob/v10.0.12/src/Hosting/Hosting/src/Internal/WebHostUtilities.cs#L7-L11) · [Construção dos endereços do host](https://github.com/dotnet/aspnetcore/blob/v10.0.12/src/Hosting/Hosting/src/GenericHost/GenericWebHostService.cs#L65-L107) · [Seleção do Kestrel](https://github.com/dotnet/aspnetcore/blob/v10.0.12/src/Servers/Kestrel/Core/src/Internal/AddressBinder.cs#L30-L60)

### P3 · O adaptador classifica interrupção da resposta como JSON inválido

O `catch` de `response.json()` devolve `invalid-json` para qualquer erro com o sinal ativo. A sondagem com o módulo real devolveu `invalid-json` tanto para um corpo cuja leitura foi interrompida quanto para JSON malformado. A tela de prontidão mostra indisponibilidade em ambos os casos. A diferença afeta a classificação técnica disponibilizada pelo adaptador a seus consumidores.

Separar falha ao ler o corpo de erro ao interpretar JSON, preservando cancelamento. Introduzido pela centralização do transporte e confirmado com respostas sintéticas. [Adaptador HTTP, linha 23](../../../scrapbook-ui/src/lib/httpClient.ts) · [Integração HTTP](../../guidelines/frontend.md#6-integração-http-e-sessão)

### Achados anteriores verificados

O Compose confirmou preservação da credencial em três falhas de preparo, recuperação da prontidão após indisponibilidade do PostgreSQL, logs sanitizados de readiness e de proxy, evento HTTP estruturado e retomada após reinício. Também confirmou os modos padrão de recusa e aceite da identidade no contêiner. Os unitários confirmaram a recuperação do fallback React e a classificação básica das respostas. A inspeção confirmou `HealthResponse` em Domain, a centralização do transporte em `lib` e a dependência de inicialização da UI pela vitalidade da API. [Runner](../../../scrapbook-tests/docker_stack.py) · [Teste do fallback](../../../scrapbook-ui/src/components/ApplicationErrorBoundary.test.tsx)

### Verificações da segunda revisão

| Verificação | Resultado |
| --- | --- |
| `git diff --check origin/dev...HEAD` nos cinco repositórios | Código `0`, sem diagnósticos. |
| `dotnet restore ScrapBook.sln --locked-mode --disable-build-servers` | Código `0`, zero avisos. |
| `dotnet build ScrapBook.sln --no-restore --no-incremental --disable-build-servers` | Código `0`, zero avisos e zero erros. |
| `dotnet format ScrapBook.sln --verify-no-changes --no-restore --severity info` | Código `0`, sem diagnósticos. |
| `dotnet test ScrapBook.sln --no-build --no-restore --disable-build-servers` | Código `0`, 32 testes aprovados, zero falhas e zero pulados. |
| `npm run lint`, `npm run build` e `npm test` | Código `0` nos três. Lint e build sem avisos, 10 testes aprovados. |
| `python3 docker_stack.py --browser` | Código `0`, cenário integrado e 10 testes Playwright aprovados, sem avisos `NO_COLOR`. |
| Sondagem isolada do adaptador HTTP | Código `0`. O reteste desativou o WebSocket incidental do Vite após o sandbox recusar sua abertura com `EPERM`. O resultado da classificação foi o mesmo nas duas execuções. |

Linhas e branches seguem sem medição neste ciclo. O AGENTS.md vigente não define percentual mínimo.
