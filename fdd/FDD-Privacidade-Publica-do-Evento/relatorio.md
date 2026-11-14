# Relatório: PP-STORY-006

## Integração posterior autorizada

Após a validação registrada abaixo, o responsável solicitou integrar todas as branches pendentes em main. PP-STORY-005 e depois PP-STORY-006 foram integradas nos cinco repositórios envolvidos, sem conflitos. [Auditoria, SHAs e checks do candidato](../../testing/2026-11-13_22-21-56/2-execucao.md). A narrativa abaixo permanece histórica. Serviços continuam parados; merge não comprova deploy nem encerra QA humano anterior.

[História](https://trello.com/c/IV6uDrPq) · [FDD](FDD-Privacidade-Publica-do-Evento.md) · [Matriz das seis histórias](matriz-de-aceite-publico.md) · [Execução](../../testing/2026-11-13_21-42-37/2-execucao.md)

## Resultado

35 subtarefas em To Deploy. [Transições nativas UTC](../../testing/2026-11-13_21-42-37/transicoes.json), sem inferir horas ativas exclusivas. [Revisões de entrega](../../testing/2026-11-13_21-42-37/revisoes.json).

A página pública continua entregando somente apresentação e disponibilidade. A consulta efetiva usa quatro colunas de Events e nenhuma identidade/contribuição. A resposta permaneceu igual antes/depois de registros privados nos três estados de participação e nos modos immediate/scheduled, inclusive com diferentes atores sintéticos.

Foram acrescentados no-referrer/nosniff, bloqueio de redirects de leitura, no-store para a SPA e logs estruturados seguros no edge. Métodos arbitrários viram OTHER nos sinais. O OpenAPI também fecha os objetos aninhados. Payload funcional, banco e dependências permanecem compatíveis.

Feature PP-STORY-006 criada da main após pull nos seis repositórios, incorporando PP-STORY-005 somente à nova feature. Promover PP-005 antes de PP-006; nenhuma main foi alterada ou publicada nesta entrega. [Base e dependências](../../testing/2026-11-13_21-42-37/dependencias-e-base.json).

## Validação

| Verificação | Resultado |
| --- | --- |
| Backend | Build sem avisos/erros e 146 unitários aprovados. |
| Frontend | Lint/build e 87 unitários aprovados. |
| Privacidade integrada | 12 comparações temporais; seis combinações ator/modo; SQL real, OpenAPI e dados permitidos; dez browser aprovados. |
| Regressões | 12 recuperação, 11 apresentação/performance, 14 gestão/health e dez compartilhamento. Total corrente: 57 browser, sem skips/retries automáticos. |
| 4G local | 739/730/749/739/725ms; máximo 749ms no perfil documentado. |
| Carga pública | 249941 consultas/60s/dez clientes, zero erros, p95 3,698ms; sem escrita. |
| Edge | 187 eventos estruturados examinados; erro 504 real com backend parado, sem URI/código/conteúdo. |

## Entrega por subtarefa

| ID | Evidência e limite |
| --- | --- |
| PP-006-001 | Lista exata de raiz, cover, janela, headers e URLs no FDD, OpenAPI e HTTP real. |
| PP-006-002 | Campos administrativos, registros, contagens, remetentes, first_record_id e URLs privadas proibidos explicitamente e ausentes. |
| PP-006-003 | Capa local pública; código/link/QR compartilháveis; contribuições/metadados privados. |
| PP-006-004 | Mesmo evento antes/depois de texto confirmado e foto provisória; payload exato com relógio fixo, sem influência no compartilhamento. |
| PP-006-005 | Código não concede ator/vínculo; testes com Bearer e X-Account-Id não ampliam acesso. |
| PP-006-006 | Contratos PP-001…005 revisados; matriz consolidada distingue resultados históricos e execução corrente. |
| PP-006-007 | DTO/modelo puro distinto da entidade administrativa; projeção reconstruída e schemas fechados inclusive aninhados. |
| PP-006-008 | Interceptor real confirma SELECT code/name/welcome_message/starts_at, sem JOIN/records/identity. |
| PP-006-009 | Serviço usa somente projeção/relógio; fixtures incluem first_record e conteúdo privado sem alterar corpo público. |
| PP-006-010 | Mesmo contrato com visitante, responsável e terceiro; sessão Cognito real permanece posterior. |
| PP-006-011 | GET/PUT/access/share existentes exigem ator/vínculo; 401/404 conforme contrato. |
| PP-006-012 | Código em PK/Bearer/header não cria permissão; sem novas rotas de mídia/galeria. |
| PP-006-013 | no-store/no-referrer/nosniff no JSON direto/proxy; condicionais retornam 200, sem cookie/ETag privado. |
| PP-006-014 | API usa rotas nominais e correlação UUID; edge sem log de URI; spans próprios sem URL/código. |
| PP-006-015 | MeterListener e unidade verificam dimensões agregadas; método desconhecido é OTHER. |
| PP-006-016 | Página guest chama apenas o endpoint público; headers completos confirmam cookies/Authorization omitidos. |
| PP-006-017 | Nenhuma busca/prefetch privado; redirect real de leitura não alcança rota de gestão. |
| PP-006-018 | DOM, atributos, modelo projetado e feedback não guardam dados administrativos/privados. |
| PP-006-019 | Storage, caches, workers, bancos locais e history sem resposta de evento; troca de código limpa apresentação anterior. |
| PP-006-020 | Nome/mensagem são texto; markup permanece literal; capa somente ativo conhecido. |
| PP-006-021 | QR/PNG/print digitais contêm apresentação pública; nome do arquivo usa código, sem identidade privada. |
| PP-006-022 | Falhas/loading/recuperação independentes de contribuições; regressão PP-005 aprovada. |
| PP-006-023 | Proxy sem cache; CORS mantém origens explícitas, escrita externa 403 e leitura sem header permissivo; logs seguros. |
| PP-006-024 | Fixtures explícitas, grafo completo, constraints ativas e cleanup exclusivo; dois textos, duas fotos provisórias e storage sintético. |
| PP-006-025 | Lista exata do OpenAPI/JSON, objetos aninhados e três estados temporais. |
| PP-006-026 | Antes/depois dos registros idênticos no mesmo evento/instante; hashes preservados após leituras. |
| PP-006-027 | Seis combinações ator/modo nos acessos diretos e proxy. |
| PP-006-028 | Tentativas de PK/código/credencial e PUT sem vínculo recusadas; fixture de payload completo corrigida. |
| PP-006-029 | Rede/DOM/storage reais em Chromium/WebKit sem marcadores privados; allHeaders inclui cabeçalhos sensíveis. |
| PP-006-030 | Navegação SPA gestão → público → outro evento sem reaproveitar dados ou solicitar API privada. |
| PP-006-031 | Cache/headers e gestão/share nos modos configurados de abertura; galeria/mídia futuras não existem e não foram declaradas testadas. |
| PP-006-032 | Logs reais API/edge e sinais próprios/SQL do probe, incluindo erro 504 e método arbitrário. Exportadores remotos não habilitados. |
| PP-006-033 | QR, PNG e print decodificados por jsQR independente; material público e filename inspecionados, sem afirmar papel/aparelho real. |
| PP-006-034 | Matriz dos cinco AC de cada história ligada aos relatórios, com execução corrente e pendências humanas explícitas. |
| PP-006-035 | FDD, revisão, base/dependência, falhas/retestes, plano/execução, limites e recuperação consolidados. |

## Limites e recuperação

[Falhas preservadas](../../testing/2026-11-13_21-42-37/falhas-e-correcoes.json). Condição de corrida no List coletor de métricas de um teste existente foi corrigida com ConcurrentBag; não desabilitou paralelismo nem enfraqueceu asserções. WebKit exigiu HTTP 302 real em loopback, pois não admite fulfill(302). CORS e payload de teste foram alinhados às regras existentes; produção não foi alterada para fazer esses testes passarem.

Leitor de tela PP-002-029 e papel/aparelhos PP-003-034 permanecem pendentes. Avaliação legal/operacional de privacidade, Cognito, uploads/S3, galeria e observabilidade remota continuam fora desta entrega. Medições locais, parcialmente simultâneas em laboratórios separados no mesmo host, não comprovam rede real ou SLA.

Sem pacote novo, mudança de schema ou DDL. Domain/Application sem nova dependência de framework. Publicar API/proxy e depois UI; voltar UI e API/proxy a versões compatíveis se necessário, preservando banco/volumes. A nova feature permanece para revisão/integração; serviços persistentes desligados e laboratórios removidos.
