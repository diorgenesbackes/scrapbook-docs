# Relatório: PP-STORY-002

[História](https://trello.com/c/rVrM2pbn) · [Contrato](FDD-Apresentacao-Publica-do-Evento.md) · [Execução](../../testing/2026-11-13_14-47-34/2-execucao.md)

## Resultado

Apresentação pública implementada em feature/PP-STORY-002: nome, capa padrão e mensagem efetiva, sem login e sem acesso a lembranças. Fallback isolado de imagem, texto Unicode seguro, tokens do guia, layout responsivo e revalidação ao voltar à aba. A resposta mantém code e acrescenta somente name/effectiveWelcomeMessage/cover. API antes da UI na publicação futura; sem nova dependência, DDL ou alteração de volume.

Antes de iniciar esta feature, o responsável autorizou parar a stack e integrar PP-STORY-001 em main. Cinco merges sem conflitos foram enviados; árvores iguais à feature validada. [Auditoria](../../testing/2026-11-13_14-47-34/integracao-PP-STORY-001.json). Esta nova US nasce dessas mains e não foi integrada automaticamente.

**Situação:** 32 subtarefas técnicas prontas para To Deploy; PP-002-029 permanece em Test e incompleta pelo teste com leitor de tela real. A história permanece em Test até essa validação. Não tratar árvore acessível como execução de VoiceOver/NVDA, nem marcar Done/deploy.

## Validação

| Verificação | Resultado |
| --- | --- |
| Backend | Build sem avisos/erros e 104 testes xUnit aprovados. |
| Frontend | Lint/build e 55 testes Vitest aprovados. |
| HTTP/DB/OpenAPI | Apresentação, edição/restauração via gestão, isolamento e projeção pública aprovados. |
| Navegador | 11 testes de apresentação/performance + seis de acesso público + 14 de gestão/health: 31 cenários distintos aprovados. |
| Página em perfil 4G | Cinco amostras: 752, 754, 751, 739 e 747ms. Máximo 754ms, abaixo de 3s. |
| Carga API | 1004 eventos, dez clientes/60s, 269199 requests, zero erros, p95 3,367ms, máximo 31,343ms. |
| Capa | SVG público existente, 823 bytes; frame 8:5 preservado na falha. |
| Somente leitura | Digests das tabelas de aplicação preservados após leituras/navegador/carga; logs amostrados sem códigos/textos. |

Perfil 4G: Chromium 158.0.8127.18, 390×844, download 10Mbps/upload 2Mbps decimais, latência mínima de headers 80ms, slowdown CPU 4×, contexto novo e cache desabilitado em cada amostra. Emulação não equivale a aparelho real/rede comercial. WebKit 27.2 para composição móvel emulada. Nenhum teste foi ignorado ou aprovado por retry automático. A regressão de gestão foi executada nos dois laboratórios, contada uma única vez no total de cenários distintos.

## Matriz de aceite

- AC01: padrão/customização/Unicode aprovados em unitários, HTTP e navegador.
- AC02: somente chave de capa permitida; fallback de erro aprovado sem perder apresentação.
- AC03: markup como texto, quebras de linha e limites preservados.
- AC04: layout/teclado/foco/texto 200% e semântica aprovados; leitor de tela real pendente, sem atestar acessibilidade integral.
- AC05: atualização/restauração de mensagem via gestão reflete na próxima consulta; revalidação da UI e isolamento entre códigos aprovados.

## Pendência PP-002-029

Executar com leitor de tela real (por exemplo VoiceOver no macOS ou NVDA no Windows), registrar leitor/versão/browser e evidência: h1 com nome correto, alt da capa, mensagem completa, status de loading/erro/revalidação, skip link/foco, ordem de leitura e texto ampliado. Não há bloqueio observado de implementação; falta a evidência humana dessa parte da subtarefa. Revisão visual integral também continua separada da inspeção local efetuada.

## Entrega por subtarefa

Estados reais ficam no Trello e em [transições](../../testing/2026-11-13_14-47-34/transicoes.json). Horários são observações UTC após operações, não horas ativas exclusivas.

| ID | Situação técnica | Entrega/evidência |
| --- | --- | --- |
| PP-002-001 | Pronta para To Deploy | Contrato dos quatro campos públicos e projeção code/name/welcome_message. |
| PP-002-002 | Pronta para To Deploy | Campos obrigatórios, mensagem não nula e fallback de capa documentados. |
| PP-002-003 | Pronta para To Deploy | Política compartilhada EventPolicy.EffectiveWelcomeMessage e testes de NULL/branco/personalização. |
| PP-002-004 | Pronta para To Deploy | Alt da capa definido; JSX seguro e white-space pre-wrap preservam texto. |
| PP-002-005 | Pronta para To Deploy | Entradas futuras de depoimento/momento registradas sem botões sem destino. |
| PP-002-006 | Pronta para To Deploy | Orçamento de 200KiB; SVG com 823 bytes e cinco amostras no perfil 4G declarado. |
| PP-002-007 | Pronta para To Deploy | PublicEventResponse, PublicEventModel e PublicEventPresentationModel puros em Domain. |
| PP-002-008 | Pronta para To Deploy | PublicEventService reutiliza a política de mensagem da gestão; sem texto padrão duplicado na UI. |
| PP-002-009 | Pronta para To Deploy | EF projeta três campos com parâmetro e AsNoTracking; sem joins de conteúdo/identidade. |
| PP-002-010 | Pronta para To Deploy | Somente referência default aprovada; sem URL assinada ou dados de mídia privada. |
| PP-002-011 | Pronta para To Deploy | OpenAPI com campos requeridos, exemplo e limites reais de texto. |
| PP-002-012 | Pronta para To Deploy | H1 usa o nome, quebra nomes longos e atualiza o título do documento. |
| PP-002-013 | Pronta para To Deploy | Frame e dimensões 960×600, proporção 8:5 mantida em loading/erro. |
| PP-002-014 | Pronta para To Deploy | Allowlist resolve o SVG padrão existente; PNGs do catálogo não foram adicionados. |
| PP-002-015 | Pronta para To Deploy | SVG vetorial responsivo, eager/high priority e dimensões reservadas. |
| PP-002-016 | Pronta para To Deploy | Falha de capa isolada; nome e mensagem continuam visíveis. |
| PP-002-017 | Pronta para To Deploy | Texto como nós React, com Unicode/quebras preservados e sem HTML executável. |
| PP-002-018 | Pronta para To Deploy | Snapshot dos tokens do guia em shared/styles e aplicação restrita ao shell guest. |
| PP-002-019 | Pronta para To Deploy | Layout em 320/390/768/1440px e texto 200% aprovados em dois browsers. |
| PP-002-020 | Pronta para To Deploy | Controles, skip link e h1 com foco/nomes acessíveis e 48px quando aplicável. |
| PP-002-021 | Pronta para To Deploy | Output de status e placeholder neutro com espaço reservado, sem nomes inventados. |
| PP-002-022 | Pronta para To Deploy | Revalidação em foco/visibilidade sem sobreposição; estado de outro código descartado. |
| PP-002-023 | Pronta para To Deploy | 104 testes backend incluindo padrão, personalização, Unicode e limites escalares. |
| PP-002-024 | Pronta para To Deploy | Vitest verifica nome de 120 caracteres e mensagem de 2000 escalares sem corte. |
| PP-002-025 | Pronta para To Deploy | Vitest e navegador verificam fallback isolado de imagem com dimensões preservadas. |
| PP-002-026 | Pronta para To Deploy | Markup exibido literalmente; nenhuma tag/handler criado ou código executado. |
| PP-002-027 | Pronta para To Deploy | HTTP modifica/restaura mensagem pela gestão e verifica a consulta pública seguinte. |
| PP-002-028 | Pronta para To Deploy | API real e composição validada nas quatro larguras em Chromium/WebKit emulado. |
| PP-002-029 | Test; parcial | PARCIAL: teclado, foco, árvore acessível e texto 200% passaram; teste com leitor de tela real pendente. |
| PP-002-030 | Pronta para To Deploy | Página pronta em 739–754ms nas cinco amostras 4G; limite de 3s atendido. |
| PP-002-031 | Pronta para To Deploy | Requests de capa só para ativo público permitido; sem galeria/mídia privada. |
| PP-002-032 | Pronta para To Deploy | Capturas/perfil/resultados e pendências humanas explicitamente registrados. |
| PP-002-033 | Pronta para To Deploy | FDD, relatório, revisões e evidências vinculados à história. |

## Limites e recuperação

Sem novas migrations, capa personalizada, QR real, janela, captura/envio, galeria, login ou AWS. Sem merge/deploy de PP-STORY-002. Reverter UI antes da API se necessário; banco preservado. A stack persistente permanece parada e os laboratórios exclusivos foram removidos. Referência de código, capturas e resultados originais estão no ciclo vinculado.
