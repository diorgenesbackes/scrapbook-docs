# Execução: PP-STORY-006

[Plano](1-plano.md) · [FDD](../../fdd/FDD-Privacidade-Publica-do-Evento/FDD-Privacidade-Publica-do-Evento.md) · [Relatório por tarefa](../../fdd/FDD-Privacidade-Publica-do-Evento/relatorio.md)

## Base e execução

[Pull nos seis projetos antes da codificação](dependencias-e-base.json), em 2026-11-14T00:38:09Z. Feature PP-006 criada diretamente de main e dependência PP-005 integrada somente na nova feature. As mains não foram alteradas nesta entrega. Publicar/promover PP-005 antes de PP-006; [revisões finais de código](revisoes.json): service 24df36d, UI c5fc91e, tests f6a0b7e e infrastructure 78c7276. Documentação em commit posterior da mesma feature.

[Checks](unitarios.json): backend build sem avisos/erros e 146 unitários aprovados; UI lint/build e 87 testes aprovados. Probe compilado sem avisos/erros. Imagens construídas por `python3 scripts/stack_local.py build`, sem iniciar a stack persistente.

## Validação de privacidade

`python3 public_privacy.py`: [resultado](public-privacy-report.json), de 01:01:08 a 01:02:33 UTC em 2026-11-14. Fixtures explícitas: dois textos confirmados, duas fotos provisórias e dois metadados de storage privados. Grafo completo de sessão/operação/Outbox e constraints habilitadas; nenhuma transferência ou arquivo S3 real.

- Doze comparações temporais antes/depois dos registros, com igualdade exata, nos estados not_open/open/closed e modos immediate/scheduled. Instante fixo pertence somente ao host de testes.
- Seis combinações de ator/modo: público 200 para todos; gestão/share 401 sem ator, 200 para responsável e 404 para terceiro. Código/Bearer/X-Account-Id não criam ator nem substituem PK; PUTs de terceiros negados.
- Lista exata de raiz, cover e submissionWindow no OpenAPI e no JSON real. API direta/proxy usam no-store/no-referrer/nosniff, sem Set-Cookie/ETag/Last-Modified de apresentação; requisição condicional continua 200.
- SQL EF real: SELECT e.code, e.name, e.welcome_message, e.starts_at FROM events.events AS e WHERE e.code = @code LIMIT 2. Sem identidade, registros, contagem ou campos administrativos.
- Probe captura span próprio events.sql.public_get e histograma de dependência com dimensões limitadas. Unitário cobre span/log/métrica HTTP, métodos arbitrários categorizados e dados de request excluídos.
- Dez testes browser aprovados em 5,6s: DOM/rede, headers completos via allHeaders, cookies omitidos apesar da fixture de cookie HttpOnly, navegação SPA de gestão para público e entre eventos, storage/cache/history, Referer ausente, redirect HTTP real recusado e QR/PNG/print digitais.
- Nginx retornou 504 ao parar somente o backend do laboratório; headers seguros e log de status/duração preservados, sem URI. Foram examinados 187 eventos estruturados de edge, com método arbitrário convertido para OTHER. Logs da API e edge amostrados sem marcadores, códigos ou URLs utilizáveis.
- Hashes de todas as tabelas de aplicação iguais após consultas, navegadores e troca de identidade. Mesmo histórico de 47 changesets, sem DDL novo.

## Regressões executadas

- `python3 public_recovery.py`: [recuperação](regressao-recuperacao.json), 12 browser aprovados, casos HTTP de link/correlação e falhas reais ACL/NOLOGIN/lock com restauração. [Resumo](public-recovery-browser-summary.json).
- `python3 public_presentation.py`: [apresentação](regressao-apresentacao.json), 11 browser de apresentação/performance e 14 de gestão/health aprovados. [Resumo público](public-presentation-browser-summary.json) e [resumo da gestão](management-browser-summary.json), derivado da saída manual do runner. Capa de 823 bytes e edição/restauração pela gestão preservadas.
- `python3 event_sharing.py`: [compartilhamento](regressao-compartilhamento.json), dez browser aprovados; dois responsáveis, terceiro, origem/Host, configuração ausente, clipboard/QR/PNG/print digitais. [Resumo](event-sharing-browser-summary.json).

Total corrente: 57 cenários browser distintos (10 privacidade + 12 recuperação + 11 apresentação/performance + 14 gestão/health + 10 compartilhamento), sem skips ou retries automáticos. Os relatórios das outras histórias preservam sua própria identificação, embora executados nesta revisão PP-006.

## Carga e performance

[Perfil 4G](performance-4g.json): Chromium 158.0.8127.18, 10/2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cache desabilitado e cinco contextos novos. Apresentação/capa/QR/janela prontos em 739/730/749/739/725ms; máximo 749ms, limite 3000ms.

Carga pública: 1004 eventos, dez clientes durante 60s, 249941 consultas, zero erros, p95 3,698ms e máximo 18,291ms. Laboratórios separados executaram parte da auditoria de privacidade e da regressão no mesmo host; não se trata de benchmark em máquina sem outra carga, rede comercial, AWS ou SLA de produção. Snapshots somente leitura conferidos.

## Falhas, limites e encerramento

[Falhas e correções](falhas-e-correcoes.json) preservam problemas de compilação/assertiva, expectativas de fetch/CORS e fixture incompleta, corrida no coletor List de métricas, limitação de route.fulfill(302) no WebKit e expectativa excessivamente específica de 502. Reteste usa HTTP 302 real em loopback, closed em finally, e registra 504 efetivo. Não atribuir falhas do runner a defeitos de autorização nem apagar execuções anteriores.

[Navegação desktop](public-privacy-navigation-chromium-desktop.png) · [Navegação WebKit](public-privacy-navigation-webkit-mobile.png). Capturas inspecionadas visualmente; mostram dados públicos sintéticos de laboratório já removido, não nomes/conteúdo de convidados.

As funções futuras de galeria/mídia e autenticação Cognito não foram implementadas. Testes de cache/autorização cobrem contratos privados existentes e modos configurados; não provam autorização temporal de endpoints futuros. Spans automáticos/exportadores remotos/ALB/WAF/Amplify não foram habilitados nem aprovados. A revisão de privacidade de lançamento permanece separada.

PP-002-029 (leitor real) e PP-003-034 (papel/aparelhos) continuam incompletas. Serviços persistentes parados, volumes preservados e laboratórios removidos. Nova feature sem merge/main/deploy. [Transições nativas do Trello](transicoes.json) são UTC e não representam horas ativas exclusivas; incluem a reabertura dos itens 007/013/025 para fechar o schema aninhado antes do reteste. As 35 subtarefas estão em To Deploy; card sem Done/deploy automático.
