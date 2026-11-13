# FDD: Acesso público ao evento

**História:** [PP-STORY-001](https://trello.com/c/LkDStr6e). Versão 0.1. Implementada/validada localmente em feature/PP-STORY-001; sem merge ou deploy.

[PRD FR-005](../../PRD.md#fr-005-página-dos-convidados-e-qr-code) · [HLD](../../HLD.md#interfaces-públicas) · [API](../../guidelines/api.md) · [S16](../../visual-identity/screens/S16.md)

## 1. Contexto e objetivos

Disponibilizar a primeira consulta anônima, somente leitura, usando o código público já gerado pela criação de eventos. Reutilizar o serviço único e o PostgreSQL existente sem resolver conta, gravar sessão ou consultar recebimentos. Meta da consulta: p95 até 500 ms, sujeita à medição local documentada.

## 2. Escopo e exclusões

Entregar URL pública, consulta mínima, shell guest, cancelamento e isolamento. Nome/capa/mensagem pertencem a PP-STORY-002, compartilhamento/QR real a PP-STORY-003 e disponibilidade a PP-STORY-004. Erros básicos seguros existem desde esta entrega; a recuperação completa é PP-STORY-005. Privacidade de PP-STORY-006 é obrigatória desde agora. Não implementar captura, envio, login, sessão de upload, galeria, novas tabelas ou publicação AWS.

## 3. Fluxo e arquitetura

1. O navegador abre /e/:code diretamente ou por navegação e carrega o shell de convidado, sem navegação de gestão.
2. O cliente faz GET /api/v1/public/events/{code} com credentials: omit e cache: no-store, sem corpo ou identidade de desenvolvimento.
3. O middleware aplica correlação, orçamento de 5s, telemetria sanitizada e no-store. Não chama ICurrentActorService no caminho público.
4. PublicEventController chama IPublicEventService; PublicEventService valida o código e consulta IPublicEventRepository.
5. PublicEventRepository usa AsNoTracking, parâmetro SQL e projeção exclusiva de code. Usa o índice único existente de events.events.code. Não usa conta, membros, registros, auditoria ou unidade de trabalho.
6. O controller retorna PublicEventResponse. O cliente confirma que o código retornado corresponde à rota e mantém somente o campo code.
7. Ao trocar de evento ou sair, AbortController cancela a chamada; callbacks antigos são descartados mesmo quando a resposta já estiver em trânsito.

Domain contém modelos/respostas/política puros. Interfaces e Services ficam em Application, EF em Infrastructure e HTTP/OpenAPI em API. Uma classe por arquivo e diretórios por conceito, sem módulos. Nenhuma dependência nova.

## 4. Contratos públicos

| Contrato | Definição |
| --- | --- |
| SPA | /e/:code; URL relativa à origem do ambiente. Não depende de domínio externo definido nesta entrega. |
| API | GET /api/v1/public/events/{code}, operationId GetPublicEvent. Sem query ou body. |
| Código | 32 bytes em base64url canônico, sem padding, 43 caracteres ASCII; A-Z, a-z, 0-9, hífen e underscore. Último caractere em AEIMQUYcgkosw048. |
| Normalização | Decodificação do segmento pelo roteador uma vez; não aplicar trim, lower/uppercase, substituir ou gerar código. Uma PK decimal não é um código válido. |
| Sucesso | 200 application/json: {"code":"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"}. Exemplo sintético, não aponta para evento cadastrado. |
| Campos permitidos | Somente code. Não retornar PK, nome, capa, mensagem, marcos, versão, permissões, remetentes, contagens ou URLs privadas nesta etapa. |
| Cancelamento | RequestAborted → serviço → EF; orçamento total 5s, command timeout existente 4s. Abandono do cliente não é erro de negócio. |
| Cache | Cache-Control: no-store na API; HTML no-cache, assets com política própria. Sem Redis, armazenamento de respostas ou service worker novo. |
| Correlação | X-Correlation-Id UUID único canônico, validado ou gerado; devolvido em sucesso/erro e exposto no CORS de origens explícitas. |
| Identidade | Não exige cookie, token, conta ou opt-in local. As rotas /api/v1/events preservam a resolução de ator e o vínculo por evento. |

O código público é compartilhável, identifica o evento e não concede autorização administrativa, de leitura de lembranças ou de envio. O contrato pode crescer de forma compatível nas próximas histórias; o cliente tolera campos adicionais, mas não os persiste.

## 5. Erros e fallback

Código inválido/não canônico e evento inexistente retornam o mesmo 404 RESOURCE_NOT_FOUND, sem ecoar o código. Query não prevista retorna 400 REQUEST_VALIDATION_FAILED. Dependência indisponível/orçamento vencido: 503 DEPENDENCY_UNAVAILABLE. Falha inesperada: 500 UNEXPECTED_ERROR. Erros usam application/problem+json com type/status/title/detail/code/correlationId e traceId real quando disponível. Não incluir instance com URL utilizável, stack trace, SQL ou dados do evento.

O shell tem loading, confirmação genérica, não encontrado e falha genérica com repetição manual de GET. Não apresenta dados de outra rota, redirecionamento para login, disponibilidade de envio ou sucesso de participação. Leitores de tela recebem output de status; o título recebe foco na abertura/troca de evento e o documento restaura o título ao sair.

## 6. Observabilidade

Operação events.public_get, dependência events.sql.public_get, rota nominal /api/v1/public/events/{code}, outcome/status/duração e correlação conforme o middleware existente. Não colocar código, URL utilizável, nome ou dados de recebimentos em logs/tags. O provedor EF mantém logging sensível desativado; logs HTTP informativos do host e access_log do Nginx seguem suprimidos. A configuração de logs de infraestrutura AWS e a auditoria transversal completa serão validadas nas entregas pertinentes; não afirmar telemetria externa instalada.

## 7. Compatibilidade, publicação e recuperação

Service e UI permanecem em feature/PP-STORY-001, base main. Publicar API antes da UI quando houver implantação futura; clientes antigos e rotas de gestão permanecem compatíveis. Não há migration, seed, nova variável de ambiente ou mudança de volume. O Nginx existente já oferece fallback de SPA e proxy /api; no Amplify futuro será necessário rewrite de rotas da SPA preservando /api e assets, sem configurar deploy agora.

Recuperação futura: restaurar a UI anterior e depois a API anterior; não desfazer dados porque a consulta não grava. Fixture e volume de validação são descartáveis e exclusivos; a stack persistente não é alterada. Não promover ou publicar branches automaticamente.

## 8. Aceite, testes e riscos

- AC01: sessão anônima abre o evento correto sem login.
- AC02: código existente é preservado, case-sensitive, e PK não é aceita como substituto.
- AC03: acesso direto, reload e nova aba funcionam em Vite e no Nginx do container.
- AC04: snapshot de todas as tabelas de aplicação permanece idêntico antes/depois de leituras.
- AC05: gestão continua negando visitante anônimo e ator sem vínculo; código não substitui identidade.

Unitários xUnit/Vitest verificam códigos, ausência de I/O inválido, portas somente leitura, cancelamento e respostas tardias. Testes manuais em scrapbook-tests cobrem HTTP, PostgreSQL, OpenAPI, navegador e carga com alvo loopback descartável. Evidências reais e falhas serão registradas em [relatório](relatorio.md). Não assumir equivalência entre emulação e aparelhos reais, nem entre carga local e infraestrutura AWS.

Riscos: o lookup público confirma existência por definição do produto, portanto códigos permanecem aleatórios de alta entropia. O shell ainda é mínimo; apresentação e QR reais dependem de outras histórias. O domínio público e o aceite visual completo seguem pendentes nas decisões existentes. A validação de sintaxe não é mecanismo de autorização.

## 9. Subtarefas rastreadas

Estado operacional e transições reais ficam no Trello; a tabela identifica escopo, sem representar conclusão antecipada.

| ID | Entrega |
| --- | --- |
| PP-001-001 | [docs] Definir a URL pública da SPA e registrar sua composição com o código base64url já existente. |
| PP-001-002 | [docs] Definir método, rota versionada, operationId e exemplos da consulta pública no contrato OpenAPI. |
| PP-001-003 | [docs] Fixar tamanho, alfabeto, sensibilidade a maiúsculas e política de encoding do código recebido. |
| PP-001-004 | [docs] Enumerar campos públicos permitidos e reservar os campos de conteúdo e janela às histórias PP-002 e PP-004. |
| PP-001-005 | [docs] Definir headers, orçamento da consulta, cancelamento e política de cache da resposta pública. |
| PP-001-006 | [service] Reutilizar o código persistido; impedir criação, rotação ou normalização destrutiva durante consultas. |
| PP-001-007 | [service/Domain] Criar o modelo puro de consulta pública com nulabilidade e conceitos explícitos, sem dependências de ASP.NET ou EF. |
| PP-001-008 | [service/Domain] Criar a resposta pública mínima com campos permitidos, separada das respostas administrativas. |
| PP-001-009 | [service/Application] Definir a porta de consulta por código público, sem parâmetros de ator ou sessão obrigatória. |
| PP-001-010 | [service/Infrastructure] Implementar busca parametrizada pelo código público, com projeção apenas dos dados necessários. |
| PP-001-011 | [service/Infrastructure] Conferir o índice único existente e o plano da busca, sem alterar changesets já aplicados. |
| PP-001-012 | [service/Application] Implementar o serviço de consulta pública sem gravações nem consulta a contribuições. |
| PP-001-013 | [service/Api] Criar controller com sufixo e diretório de conceito e acesso anônimo limitado à nova operação. |
| PP-001-014 | [service/Api] Aplicar validação do código, CancellationToken, correlação e contrato de resposta. |
| PP-001-015 | [service/Api] Registrar os componentes na composição preservando as regras de autorização das rotas privadas. |
| PP-001-016 | [ui] Registrar a rota pública sem depender do estado de login ou da identidade local de desenvolvimento. |
| PP-001-017 | [ui] Criar cliente HTTP tipado para a consulta pública, separado do contrato de gestão. |
| PP-001-018 | [ui] Implementar carregamento inicial e cancelamento ao trocar de rota ou evento. |
| PP-001-019 | [ui] Impedir que respostas antigas de outra navegação substituam o evento atual. |
| PP-001-020 | [ui] Compor o shell guest de S16, com título de página e foco inicial acessível. |
| PP-001-021 | [infrastructure] Ajustar, se necessário, o fallback da SPA para acesso direto à nova URL, preservando o encaminhamento de /api. |
| PP-001-022 | [infrastructure] Documentar URLs locais e configuração de hospedagem necessárias, sem implantar serviços. |
| PP-001-023 | [service/UnitTests] Cobrir código válido, diferença de maiúsculas e formato inválido conforme o contrato. |
| PP-001-024 | [service/UnitTests] Cobrir serviço com evento encontrado e não encontrado e verificar ausência de comandos de escrita. |
| PP-001-025 | [ui/test] Cobrir abertura sem sessão, troca de código na rota e descarte de resposta fora de ordem. |
| PP-001-026 | [tests/HTTP] Consultar evento válido sem cookie, token, header de identidade local ou conta cadastrada. |
| PP-001-027 | [tests/HTTP] Verificar que a PK e o código de outro evento não permitem acessar gestão administrativa. |
| PP-001-028 | [tests/PostgreSQL] Comparar o estado persistido antes/depois da consulta e validar isolamento entre eventos. |
| PP-001-029 | [tests/E2E] Abrir link em contexto anônimo e validar reload, nova aba e acesso direto no container da UI. |
| PP-001-030 | [tests/performance] Medir consulta por código com volume e concorrência documentados e comparar p95 com a meta de 500 ms. |
| PP-001-031 | [docs] Registrar contrato final, comandos, resultados, riscos e dependências entre repositórios. |
| PP-001-032 | [docs] Vincular evidências e revisão à história, com horários reais e nenhuma alegação de deploy automático. |
