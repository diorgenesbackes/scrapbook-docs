# FDD: Apresentação pública do evento

**História:** [PP-STORY-002](https://trello.com/c/rVrM2pbn). Versão 0.1, implementada/validada localmente em feature/PP-STORY-002; leitor de tela real pendente.

[PRD FR-005](../../PRD.md#fr-005-página-dos-convidados-e-qr-code) · [HLD](../../HLD.md#interfaces-públicas) · [S16](../../visual-identity/screens/S16.md) · [PP-STORY-001](../FDD-Acesso-Publico-do-Evento/FDD-Acesso-Publico-do-Evento.md)

## 1. Contexto e objetivos

Conectar a configuração existente do evento à apresentação pública sem login. A base PP-STORY-001 foi integrada em main por solicitação do responsável antes de criar esta feature em service, UI, tests e docs. Serviços persistentes permanecem parados; a validação usa volumes descartáveis. Não integrar automaticamente esta nova história em main.

## 2. Escopo e exclusões

Nome, capa padrão, mensagem efetiva, fallback isolado da capa, renderização segura, tokens, responsividade e revalidação no retorno à aba. Não implementar capa personalizada, QR, janela de envios, nome/concordância do convidado, captura/envio, galeria, autenticação ou AWS.

Entradas futuras Deixe um depoimento e Registre um momento dependem dos fluxos S17/S18/S19 e respectivas histórias de participação. Não mostrar botões sem destino, simular envio nem inferir disponibilidade antes de PP-STORY-004. PP-STORY-003 implementará link e QR real.

## 3. Fluxo e fronteiras

PublicEventRepository projeta exclusivamente code, name e welcome_message com AsNoTracking e parâmetro SQL. Não carrega entidade inteira, membros, registros, remetentes, contagens ou mídia privada. PublicEventService converte o modelo consultado em PublicEventPresentationModel: reutiliza EventPolicy.EffectiveWelcomeMessage e a chave constante de capa aprovada. EventDetailsModel utiliza a mesma política, sem mudança de comportamento na gestão.

Modelos, respostas e mapper puros ficam em Domain; serviço e portas em Application, EF em Infrastructure e controller/OpenAPI na API. Sem novas referências, migrations ou seeds. GET continua sem escrita, identidade obrigatória ou unidade de trabalho.

A UI mantém apenas os campos permitidos, apresenta h1, imagem e mensagem como nós de texto e cancela callbacks antigos. Revalida no foco/retorno à visibilidade, sem polling; ref de requisição impede duplicação de eventos focus/visibility. Revalidação do mesmo evento pode preservar a apresentação durante loading; falha remove o dado antigo e permite novo GET manual. Troca de código nunca exibe o evento anterior. Sem armazenamento em localStorage/sessionStorage.

## 4. Contrato público

A rota permanece GET /api/v1/public/events/{code}, operationId GetPublicEvent; SPA /e/:code. Código, erros, correlação, cancelamento, orçamento de 5s, no-store e proteção da gestão permanecem conforme PP-STORY-001.

| Campo | Regra |
| --- | --- |
| code | Código existente, canônico, 43 caracteres, case-sensitive. |
| name | Obrigatório; nome configurado, 1–120 valores escalares Unicode conforme regra vigente. |
| effectiveWelcomeMessage | Obrigatório, não nulo, até 2000 valores escalares Unicode; personalizado normalizado ou padrão do servidor. |
| cover | Obrigatório; somente kind=default e assetKey=event-cover-default-v1 nesta versão. Nunca uma URL recebida de contribuições. |

Exemplo: {"code":"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA","name":"Encontro de amigos","effectiveWelcomeMessage":"Que bom ter você aqui!\\nGuarde um momento com carinho.","cover":{"kind":"default","assetKey":"event-cover-default-v1"}}. Código sintético, não um evento cadastrado.

Texto padrão vigente: Seja bem-vindo! Compartilhe uma lembrança deste evento. NULL no banco usa o padrão; JSON nulo/vazio/branco na gestão já é normalizado para NULL. A UI não copia o texto padrão nem normaliza novamente o texto efetivo. Preservar quebras de linha via white-space: pre-wrap e renderizar strings em JSX, sem innerHTML/HTML/Markdown executável.

Não retornar PK, welcomeMessage bruto, versão, permissões, marcos, registros, contagem, remetentes ou URL privada. Evolução aditiva: o cliente anterior que usa code tolera campos extras. A UI nova exige a apresentação, portanto a API deve ser publicada antes.

## 5. Capa e estados visuais

A allowlist local resolve somente a referência conhecida para /assets/event-cover-default-v1.svg, já existente. SVG vetorial 960×600, proporção 8:5; não recortar e não criar variantes raster desnecessárias. width/height e frame com aspect-ratio reservam dimensões no loading/erro; a capa usa eager/high priority por ser o hero. Alt: Ilustração de uma paisagem na capa padrão do evento.

Referência desconhecida/ausente ou erro da imagem produz Capa indisponível., preservando nome/mensagem. Nenhuma referência privada é convertida em src. Loading tem área neutra sem fotografias ou nomes inventados; erro de consulta e 404 mantêm os estados básicos anteriores.

Tokens do guia são copiados para shared/styles/tokens.css e usados com escopo guest; não importar código do catálogo nem PNGs pesados. Composição de uma coluna até 42rem, a partir de 320px. Texto longo quebra sem rolagem horizontal; controles mantêm 48px. Nome do evento atualiza título do documento; o foco não salta durante revalidação. Output anuncia estado; imagem e h1 têm semântica explícita.

## 6. Orçamentos e observabilidade

Capa mobile até 200KiB (orçamento proposto no handoff, adotado para esta entrega); a versão SVG existente deve permanecer abaixo desse limite. Medir página pronta quando nome, mensagem e capa real estão visíveis/carregados.

Perfil local de validação 4G: Chromium 158, viewport 390×844, 10Mbps download/2Mbps upload em unidades decimais, latência mínima de headers 80ms e slowdown de CPU 4× via CDP. Cinco contextos novos, cache desabilitado; cada amostra deve ficar até 3000ms a partir da navegação. Perfil declarado, sem alegar equivalência a aparelho real ou rede comercial. Meta da API p95 até 500ms medida separadamente com 1004 eventos, dez concorrentes/60s.

Operações events.public_get e events.sql.public_get permanecem, sem código/nome/mensagem em logs, traces ou labels. Registro de performance só agrega tempos/perfil, sem links utilizáveis. Telemetria AWS e auditoria transversal completa continuam nas entregas próprias.

## 7. Compatibilidade e recuperação

API antes da UI; clientes anteriores preservam leitura de code. Reverter UI antes da API em recuperação. Sem banco/migration, dependência nova, configuração externa ou mudança de volume. Branch iniciada da main com cinco merges de PP-STORY-001 preservando árvores validadas. Repositórios infrastructure/liquibase não exigem nova feature.

## 8. Aceite e testes

- AC01: nome e mensagem efetiva configurados, com padrão calculado no servidor.
- AC02: capa pública permitida e fallback isolado.
- AC03: texto seguro com Unicode, acentos e quebras de linha.
- AC04: composição desde 320px, teclado, foco, texto 200% e semântica acessível.
- AC05: edições permitidas refletidas na próxima consulta, sem consultar recebimentos.

Unitários xUnit/Vitest; HTTP/PostgreSQL, navegador, performance e carga manuais em scrapbook-tests. Testes HTTP alteram/restauram a mensagem pela API de gestão e comparam a próxima resposta pública. Browser usa API real, registra árvore acessível, teclado/foco e dimensões em Chromium/WebKit emulado. A árvore semântica não equivale a leitor de tela real; registrar essa parte de PP-002-029 como pendência de validação humana, sem declarar conclusão integral do item.

[Relatório e evidências](relatorio.md) identificarão revisões, falhas/retestes, comandos, limitações e estados por subtarefa.

## 9. Riscos e decisões

A capa atual é a ilustração padrão já existente; não substituir automaticamente por fotografia sintética do catálogo. Implementação completa de capas/identidade visual mantém as questões Q02/Q07 do guia. Não habilitar navegação de contribuição enquanto os destinos não existirem. O contrato público pode revelar nome/mensagem por definição do produto, sem ampliar acesso a lembranças. Domínio, implantação e aceites humanos não são inferidos dos testes locais.

## 10. Subtarefas

| ID | Entrega |
| --- | --- |
| PP-002-001 | [docs] Mapear nome, capa pública e mensagem efetiva da configuração existente para a resposta pública. |
| PP-002-002 | [docs] Definir campos obrigatórios, nulabilidade e comportamento de ausência de capa ou mensagem. |
| PP-002-003 | [docs] Confirmar o texto padrão vigente e a regra de precedência da mensagem personalizada. |
| PP-002-004 | [docs] Especificar alt da capa e preservação de quebras de linha sem aceitar HTML do usuário. |
| PP-002-005 | [docs] Registrar os pontos de entrada futuros de depoimento/momento e suas dependências, sem assumir os fluxos implementados. |
| PP-002-006 | [docs] Definir orçamento da capa mobile e perfil de medição, distinguindo proposta visual de meta validada. |
| PP-002-007 | [service/Domain] Estender a resposta pública com nome, mensagem efetiva e referência permitida da capa. |
| PP-002-008 | [service/Application] Reutilizar a regra existente de boas-vindas sem duplicar o texto padrão no frontend. |
| PP-002-009 | [service/Infrastructure] Projetar os campos de apresentação sem joins com registros, remetentes ou mídias privadas. |
| PP-002-010 | [service/Application] Resolver somente referências de capa pública aprovadas, sem produzir URL assinada de lembranças. |
| PP-002-011 | [service/Api] Atualizar os exemplos e schemas OpenAPI com os limites reais dos textos existentes. |
| PP-002-012 | [ui] Implementar o título do evento com hierarquia semântica e quebra de nomes longos. |
| PP-002-013 | [ui] Implementar a capa com proporção e dimensões reservadas para evitar deslocamento do layout. |
| PP-002-014 | [ui] Resolver a capa padrão existente por contrato, sem copiar as imagens sintéticas pesadas do catálogo para produção. |
| PP-002-015 | [ui] Aplicar carregamento e dimensionamento responsivos da imagem conforme o ativo disponível. |
| PP-002-016 | [ui] Implementar fallback visual acessível para falha da capa, sem bloquear o restante da página. |
| PP-002-017 | [ui] Renderizar mensagem efetiva como texto seguro, com suporte às quebras de linha previstas. |
| PP-002-018 | [ui] Aplicar tokens de cores, tipografia, espaçamento e componentes do guia visual. |
| PP-002-019 | [ui] Ajustar composição de S16 para 320/390 px e para tablet/desktop sem rolagem horizontal. |
| PP-002-020 | [ui] Garantir que controles e links existentes tenham nome acessível e foco visível. |
| PP-002-021 | [ui] Anunciar carregamento da apresentação e preservar dimensões estáveis durante a troca de estado. |
| PP-002-022 | [ui] Revalidar a apresentação ao retornar à página sem reutilizar indefinidamente dados de outro evento. |
| PP-002-023 | [service/UnitTests] Cobrir mensagem padrão, mensagem personalizada e conteúdo Unicode conforme normalização existente. |
| PP-002-024 | [ui/test] Cobrir nome longo, mensagem no limite permitido e quebras de linha. |
| PP-002-025 | [ui/test] Cobrir carregamento, sucesso e erro isolado da imagem com fallback acessível. |
| PP-002-026 | [ui/test] Validar que textos contendo markup são exibidos como texto e não executados. |
| PP-002-027 | [tests/HTTP] Conferir que a resposta pública reflete alterações de apresentação permitidas na gestão. |
| PP-002-028 | [tests/E2E] Validar S16 em 320, 390, 768 e 1440 px com os dados reais da API local. |
| PP-002-029 | [tests/a11y] Verificar teclado, foco, leitor de tela e ampliação de texto a 200% na apresentação. |
| PP-002-030 | [tests/performance] Medir página pronta no perfil 4G documentado e comparar com a meta de 3 segundos. |
| PP-002-031 | [tests/privacy] Inspecionar requests da capa e garantir que nenhuma mídia de lembrança é solicitada. |
| PP-002-032 | [docs] Registrar screenshots, perfil de rede, resultados, desvios visuais e revisão humana pendente ou realizada. |
| PP-002-033 | [docs] Atualizar o contrato de apresentação e vincular evidências à história. |
