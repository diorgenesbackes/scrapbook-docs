# Guideline de frontend

[Índice](README.md) · [Princípios](principios.md) · [API](api.md) · [Mensageria e mídias](mensageria.md)

Aplicação: React e TypeScript, frontend estático previsto no Amplify, prioritariamente móvel e em português do Brasil. React 19.3.1, React Router 8.5.1, TypeScript 7.0.2, Vite 8.3.6, Oxlint 1.90.0 e Vitest 5.1.0/Testing Library/jsdom estão fixados. Node 26.13.0/npm 11.20.0 são as versões validadas. Consulte [execução e configuração do frontend](https://github.com/diorgenesbackes/scrapbook-ui/blob/main/README.md) e lockfile. Playwright 1.65.0 é manual em scrapbook-tests; publicação ainda não configurada.

## 1. Organização de pastas e responsabilidades

A estrutura abaixo é o padrão adotado para o reinício do projeto, conforme a definição do responsável. As pastas de uso global separam responsabilidades compartilhadas; `pages` e `features` agrupam o código exclusivo de cada página ou funcionalidade. Criar cada pasta quando houver conteúdo e responsabilidade definidos.

```text
src/
  assets/                    # imagens, fontes e outros recursos da aplicação
  components/                # componentes reutilizáveis de uso global
  config/                    # configurações globais e públicas do frontend
  context/                   # contextos e providers de estado compartilhado
  features/                  # funcionalidades que precisam de organização própria
  hooks/                     # hooks personalizados de uso global
  layouts/                   # estruturas visuais compartilhadas entre páginas
  lib/                       # configuração e adaptação de bibliotecas externas
  pages/                     # páginas da aplicação e seu código exclusivo
  routes/                    # definição e composição das rotas
  services/                  # consumo de serviços e integrações de uso global
  styles/                    # estilos globais, temas e tokens
  utils/                     # funções utilitárias puras e reutilizáveis
  App.tsx                    # composição dos providers e das rotas
  main.tsx                   # entrada da aplicação
  vite-env.d.ts              # tipagem do ambiente Vite
```

Padronizar os nomes como na árvore: `config` e `context` no singular; `assets`, `components`, `features`, `hooks`, `layouts`, `pages`, `routes`, `services`, `styles` e `utils` no plural. Pastas de páginas e funcionalidades usam nomes em inglês com `kebab-case`, como `event-detail` e `guest-identification`.

### Responsabilidade de cada pasta

| Pasta | Conteúdo e critério de uso | Exemplos |
| --- | --- | --- |
| `assets` | Recursos estáticos utilizados pela aplicação, como imagens, ícones e fontes. Mocks estáticos podem servir como fixtures de teste, com dados sintéticos e importação restrita aos testes. | `assets/images`, `assets/fonts`, `assets/mocks` |
| `components` | Componentes com contrato reutilizável por diferentes áreas da aplicação. Componentes exclusivos de uma página ou feature ficam no respectivo escopo. | `Button`, `Modal`, `LoadingIndicator` |
| `config` | Configurações e constantes globais: leitura validada do ambiente, opções públicas e valores padrão. Tudo que vai para o bundle é público. | `environment.ts`, `applicationConfig.ts` |
| `context` | Contextos e providers para compartilhar e manipular estado entre partes da aplicação. Usar apenas quando o estado precisar ser compartilhado nesse alcance. | `ThemeContext.tsx`, `SessionContext.tsx` com estado de sessão permitido |
| `features` | Código de uma capacidade de negócio com organização própria: componentes, serviços, hooks, modelos e testes exclusivos. Usar em projetos grandes, funcionalidades complexas ou com perspectiva de crescimento. | `features/events`, `features/guest-identification` |
| `hooks` | Hooks personalizados reutilizados globalmente. Hooks de uma página ou feature permanecem nesse escopo. | `useOnlineStatus.ts`, `useMediaQuery.ts` |
| `layouts` | Estruturas visuais que envolvem páginas, como navegação, cabeçalho, rodapé e área de conteúdo. | `OwnerLayout.tsx`, `GuestLayout.tsx` |
| `lib` | Configuração e adaptação de bibliotecas externas: transporte HTTP, configuração de um SDK ou wrapper necessário à integração. | `httpClient.ts` baseado em fetch ou na biblioteca adotada |
| `pages` | Páginas acessadas pelas rotas e composição da experiência de cada tela. Componentes, hooks, estilos e serviços exclusivos da página podem ficar junto dela. | `pages/event-detail/EventDetailPage.tsx` |
| `routes` | Definição de caminhos, configuração do roteador, associação entre páginas/layouts e guards de navegação. Autorização de negócio continua no backend. | `AppRoutes.tsx`, `routePaths.ts` |
| `services` | Operações que consomem APIs ou serviços externos e são utilizadas globalmente. Usam os adaptadores de `lib`; operações exclusivas ficam na página ou feature. | `analyticsService.ts`, respeitando as regras de privacidade |
| `styles` | Estilos que se aplicam globalmente: reset, tipografia, temas e tokens. Estilos exclusivos acompanham o componente, página ou feature. | `global.css`, `tokens.css`, `theme.css` |
| `utils` | Funções puras de uso geral, sem hooks, estado React, chamadas HTTP ou dependência de uma tela. | `formatCurrency.ts`, `formatBusinessDate.ts` |

Recursos que precisam de URL pública estável podem ficar em `public`, conforme o bundler. Fotos, áudios e vídeos enviados pelos convidados continuam no armazenamento privado; fixtures e mocks usam somente dados sintéticos. Setup e helpers compartilhados de testes podem ficar em `src/test`; unitários ficam ao lado do código testado.

### Escolher entre global, página e feature

O alcance da responsabilidade determina a localização:

- Uso exclusivo de uma página: `pages/<pagina>/components`, `hooks`, `services` ou arquivos próximos da página.
- Uso exclusivo de uma funcionalidade: `features/<feature>/components`, `hooks`, `services` e demais arquivos necessários.
- Uso global: a pasta da raiz correspondente à responsabilidade, como `components`, `hooks`, `services` ou `utils`.

Uma feature reúne uma capacidade de negócio que pode atender várias páginas, como gestão de eventos. Em uma aplicação simples, a organização por páginas e pastas globais é suficiente. Introduzir `features` quando essa separação ajudar a manter ou evoluir a funcionalidade. Extrair código para o alcance global quando houver uma responsabilidade compartilhada clara, preservando contratos pequenos.

Exemplo de uma página que compõe uma feature e mantém seu componente exclusivo:

```text
src/
  pages/
    event-detail/
      EventDetailPage.tsx
      EventDetailPage.test.tsx
      components/
        EventDetailHeader.tsx
  features/
    events/
      components/
        EventForm.tsx
        EventForm.test.tsx
        EventForm.css
      hooks/
        useEvent.ts
      services/
        eventService.ts
      eventModel.ts
      index.ts
```

O `index.ts` de uma feature, quando útil, expõe somente sua interface pública. Os diretórios internos são criados conforme o tamanho da funcionalidade, sem repetir toda a estrutura global em cada feature.

### Direção das dependências

`main.tsx` inicializa a aplicação; `App.tsx` compõe providers globais e rotas; `routes` associa páginas a layouts; `pages` compõe funcionalidades e componentes. Manter regras de negócio, transporte e detalhes de bibliotecas nos arquivos responsáveis por essas funções.

- Código reutilizável em `assets`, `components`, `config`, `context`, `hooks`, `lib`, `services`, `styles` e `utils` não importa páginas, detalhes de features, rotas ou `App.tsx`.
- Layouts usam componentes e providers globais, além das primitivas do roteador para exibir o conteúdo. A associação entre um layout e uma página pertence a `routes`.
- Features podem usar recursos globais e não dependem de páginas ou da composição de rotas. Colaboração entre features usa interfaces públicas pequenas e dependências explícitas, sem ciclos.
- Componentes globais recebem dados e ações por props; requisições específicas de uma funcionalidade permanecem no serviço/hook dessa funcionalidade.
- `lib` configura o mecanismo de integração; `services` oferece as operações de consumo; `hooks` integra lógica com o ciclo de vida React; `utils` contém funções puras.

Componentes e páginas usam `PascalCase`; hooks usam `use` seguido de inicial maiúscula, como `useEvent`. Funções utilitárias usam nomes que descrevem a ação. Arquivos de teste usam `*.test.ts` ou `*.test.tsx`, próximos da implementação.

### Aplicação ao código existente

Esta atualização define o padrão de organização; a implementação anterior ainda contém `app` e `shared`. Na reorganização do frontend, distribuir a composição de `app` entre `App.tsx`, `routes`, `layouts`, `context` e `styles`; distribuir `shared` entre as pastas globais segundo sua responsabilidade. Por exemplo, `shared/api/http.ts` corresponde ao adaptador em `lib`, e `shared/ui` corresponde a `components`. Funções puras de `shared/lib` pertencem a `utils`; adaptadores de bibliotecas pertencem a `lib`.

Preservar os contratos, comportamento, testes e regras de privacidade ao mover arquivos. Os nomes de entrada e tipagem acompanham o tooling adotado: o projeto atual usa Vite, `main.tsx` e `vite-env.d.ts`. A [organização documentada no frontend](https://github.com/diorgenesbackes/scrapbook-ui/blob/main/README.md#organização) distingue a estrutura anterior desta estrutura-alvo.

Componentes não conhecem schema SQL ou SDK de backend. O adaptador converte respostas nos modelos da tela, sem reproduzir as quatro camadas .NET no React.

### Identidade visual proposta

A [especificação visual](../visual-identity/README.md) de feature/3 cobre todas as jornadas do PRD. Direção escolhida: fotográfico minimalista, fundos neutros e foco nas mídias. Tokens CSS, componentes, medidas, estados, navegação e exemplos são reutilizáveis, mas a proposta aguarda revisão do responsável e decisões de produto/provedor em aberto. Não presumir aprovação visual nem aplicar novas capacidades a partir do catálogo. As telas gerenciadas Cognito exigem validação no provedor.

## 2. Componentes, composição e hooks

- Renderização pura; não gravar, iniciar upload ou alterar props durante render.
- Estado atualizado por novas referências; composição preferida a herança.
- Hooks chamados segundo suas regras, com cleanup de subscriptions, timers e captura.
- Chaves estáveis em listas; não usar índice para itens reordenáveis nem gerar UUID a cada render.
- Extrair componente/hook quando houver responsabilidade ou reutilização real.
- Não tornar uma função pura em hook sem necessidade.

Hooks personalizados compartilham lógica. O estado local criado dentro de um hook pertence a cada chamada; extrair um hook não compartilha automaticamente uma mesma instância de estado. Para compartilhar estado, definir seu proprietário e usar props ou um provider no alcance adequado. Seguir a [documentação atual de hooks personalizados do React](https://react.dev/learn/reusing-logic-with-custom-hooks).

Captura, revisão, upload e galeria são interações distintas. Uma legenda compartilhada por um conjunto não exige manter cópias editáveis independentes em cada foto.

## 3. Estado e ciclo de vida

| Informação | Local e cuidados |
| --- | --- |
| Campo, diálogo, etapa de captura | Estado local ou reducer da jornada. |
| Estado compartilhado entre partes da aplicação | Provider em `context`, ou contexto local à página/feature quando esse for seu alcance. |
| Arquivo/blob e rascunho | Memória da página; não prometer recuperação após recarga/saída. |
| Nome e concordância | Persistência no navegador por evento; alterar nome exige nova concordância. |
| Filtro/posição navegável | URL quando apropriado, sem tokens ou conteúdo privado. |
| Evento, sessão e galeria | Camada de consulta com invalidação e identidade corretas. |
| Progresso e tentativa lógica | Estado da operação; identidade estável durante retry. |
| Contagem regressiva | Derivação da data e referência temporal do servidor. |

Evitar estados contraditórios e cópias redundantes. Usar uniões discriminadas para `idle`, `capturing`, `reviewing`, `uploading`, `validating`, `confirmed` e `error`, ajustadas ao contrato efetivo.

Biblioteca de estado/cache só quando houver necessidade. Se houver cache, incluir usuário, evento, filtros e autorização na identidade; limpar dados privados ao sair, trocar conta/evento e mudar para bloqueio. Não persistir lembranças, URLs assinadas ou tokens em armazenamento local.

Nome/concordância local é conveniência, não prova de identidade nem autorização. O conteúdo em andamento permanece apenas enquanto a página o conservar.

## 4. Effects e assincronismo

Effects sincronizam sistemas externos; cliques em Enviar, Reproduzir e Próximo disparam ações do usuário. Não iniciar gravação ou upload por flag observada em Effect quando a ação pertence ao evento.

Cancelar consultas antigas com `AbortController` e/ou ignorar respostas obsoletas. Troca de evento, logout ou bloqueio deve impedir que uma resposta anterior reexiba mídia. Declarar dependências reais e corrigir loops em vez de silenciar lint.

Parar trilhas de câmera/microfone e liberar object URLs quando deixarem de ser necessárias. Limpar timers/polling ao desmontar ou concluir. O ciclo extra do StrictMode em desenvolvimento não deve duplicar envio ou deixar captura ativa.

## 5. TypeScript e contratos

Adotar `strict: true` desde a criação do projeto. A base também habilita `noUncheckedIndexedAccess` e `exactOptionalPropertyTypes` na aplicação e nas configurações; essas opções são explícitas e não estão implícitas em `strict`.

- Usar `unknown` nas fronteiras e validar entradas/respostas críticas.
- `as Response` não valida JSON em runtime.
- Evitar `any`, `!` e dezenas de campos opcionais para esconder estados incompatíveis.
- Definir enums, ausência, nulabilidade, bytes, segundos, caracteres e datas conforme a API.
- Tipos gerados do OpenAPI exigem ferramenta fixada e regeneração reproduzível.
- IDs são opacos; não extrair permissão do formato ou da sequência. IDs bigint chegam como strings decimais e não devem ser convertidos para Number.

## 6. Integração HTTP e sessão

Centralizar base URL, headers, credenciais, cancelamento e interpretação de erros no adaptador HTTP em `lib`. Serviços de consumo usam esse adaptador e validam os contratos da operação. Serviços/contratos exclusivos ficam junto da página ou feature; `services` na raiz atende operações de uso global.

Seguir [API](api.md): prefixo `/api/v1`, sem envelope universal, cursor e `204` sem leitura de JSON. Verificar status de `fetch`; distinguir falha de rede, cancelamento, HTTP e JSON inválido. Erros de ALB/WAF/S3 podem não ser JSON.

`401` conduz à sessão; `403` apresenta restrição aplicável; `409` reconcilia estado sem sobrescrever; `429/503` apresentam recuperação segura. Usar códigos estáveis, mensagem em português e correlação para suporte.

Login é iniciado pelo backend e usa telas gerenciadas Cognito. O frontend redireciona ao fluxo gerenciado. Tokens Cognito não ficam no JavaScript, URL, console, `localStorage` ou `sessionStorage`. O navegador recebe cookie seguro HttpOnly e consulta apenas o estado permitido.

CSRF, CORS e credenciais devem ser definidos junto dos domínios e callback OIDC. Não presumir que mesmo domínio, SameSite ou ocultação de botão resolvam autorização.

## 7. Captura, revisão e envio

- Oferecer seleção de arquivo quando captura/permissão falhar.
- Mostrar tempo restante, encerrar gravação no limite e levar à revisão; nunca enviar automaticamente.
- Aplicar limites do [guia de mídias](mensageria.md), com backend como autoridade.
- Depoimentos usam um formato por envio; Momentos usam até 10 fotos ou um vídeo.
- Legenda opcional aplica-se ao conjunto; textos/legendas respeitam 5.000/500 caracteres.
- Contador de texto surge quando restarem 500 caracteres; de legenda, 100. Limitar digitação/colagem preservando edição.
- Revisão permite refazer, trocar, editar e descartar antes do envio.
- Ações de saída controladas mostram “Você pode perder o trabalho em andamento.”; fechamento/recarga pode usar aviso nativo, sem prometer que será sempre exibido.

Ao tocar em Enviar, solicitar sessão com chave idempotente e enviar diretamente ao S3. Preservar a mesma tentativa/chave na recuperação de resposta perdida; não gerar nova a cada retry.

Mostrar progresso real da transferência em porcentagem e separar a etapa posterior de validação. Não apresentar 100% de bytes como confirmação. Bloquear novo acionamento e outro registro enquanto a tentativa estiver em andamento.

Vídeos grandes usam multipart e repetem partes falhas. Falhas preservam conteúdo disponível na página para tentar novamente dentro do prazo. Não prometer upload offline ou retomada após recarga.

Consultar status a cada 5 segundos enquanto pendente, sem requisições sobrepostas; encerrar no estado terminal. O status pode ser consultado sem conceder leitura do conteúdo. Depois de confirmado, liberar recursos locais da prévia e mostrar apenas confirmação e próximas ações permitidas.

Se o servidor concluir que o prazo terminou, mostrar “O prazo para concluir este envio terminou.”. O relógio local não reserva a janela e a tolerância até 13h não permite iniciar tentativa nova depois do meio-dia.

## 8. Datas, bloqueio e visualização

Mostrar horários em Brasília, sem seletor de fuso. Parse de datetime local precisa de semântica explícita; não depender de `new Date` no timezone do dispositivo. Usar referência temporal do servidor para contagem regressiva, mantendo autorização na API.

Antes da abertura, mostrar somente data e contagem regressiva em dias, horas e minutos. Não buscar/pré-carregar galeria, número de registros, remetentes, miniaturas ou mídia para ocultar com CSS.

Na mudança para bloqueio, interromper reprodução, cancelar consultas pendentes e limpar URLs/conteúdo sob controle da aplicação. Impedir novas autorizações. URLs anteriores têm tolerância de até 2 minutos e conteúdo já entregue não pode ser recolhido.

Galeria usa cursor e até 10 itens, filtros Depoimentos/Momentos/Tudo e ordem cronológica. Fotos do conjunto aparecem separadamente com mesmo nome/legenda.

Áudio/vídeo iniciam somente por Reproduzir; fim da mídia não avança. Próximo exige ação explícita. Não adicionar busca por nome, download, edição, exclusão ou autoplay.

Mídia em processamento/falha é sinalizada somente ao responsável com acesso liberado. Renovar URL expirada exige nova autorização, sem fallback público.

## 9. Formulários, acessibilidade e compatibilidade

Usar HTML semântico, labels associados, foco visível, mensagens de campo e resumo de erros quando necessário. Diálogos administram foco e retornam ao acionador. Progresso e estados devem ser acessíveis sem anúncios repetidos a cada render.

Meta WCAG 2.2 AA: verificar teclado, contraste, ampliação, leitores de tela e informação sem dependência exclusiva de cor/ícone. Acessibilidade das mídias dos convidados precisa de revisão própria; uma interface acessível não comprova conformidade de todo o conteúdo.

Toda tela remota trata carregamento, vazio autorizado, erro e sucesso; escrita inclui pendência. Estado bloqueado não é galeria vazia. Preservar campos válidos após falha.

### Recuperação de falhas da interface

**DEVE:** definir Error Boundary na composição da aplicação e fronteiras por rota/jornada quando houver recuperação independente. O fallback mostra mensagem segura, correlação quando disponível e ação explícita para tentar novamente ou voltar a uma rota conhecida. A [documentação do React sobre Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) descreve essa proteção para falhas de renderização.

Erros de handlers de eventos e operações assíncronas continuam tratados no fluxo da ação/consulta; Error Boundary não substitui o adaptador HTTP nem a máquina de estados de upload. Tratar também falha de carregamento de módulos/rotas, inclusive após uma publicação, sem loops de retry ou recarga automática.

Resetar a fronteira não pode repetir envio, gerar nova chave idempotente ou misturar conta/evento anterior. Liberar câmera/microfone, timers e consultas do trecho desmontado. Se a recuperação exigir recarregar e houver conteúdo em memória, informar a possível perda e aguardar ação do usuário. Não prometer restaurar blobs descartados; confirmar estado no servidor antes de repetir uma escrita com resultado desconhecido.

Validar Android, iPhone e desktop nas duas versões principais mais recentes dos navegadores previstos no PRD, registrando versões efetivamente testadas. Testar permissão negada, captura indisponível, orientação, conteúdo longo e rede instável. Incluir pessoas de diferentes idades nos testes do protótipo.

## 10. Segurança, desempenho e dependências

Não renderizar texto não confiável como HTML. Não registrar depoimento, legenda, nome, blob, tokens ou URLs utilizáveis em telemetria. Link/QR code público permite participação no evento, não leitura das lembranças.

Configuração incluída no bundle é pública. Se Vite for adotado, `VITE_*` nunca contém segredos. Variável de runtime não altera automaticamente arquivo estático já compilado.

Priorizar a página dos convidados pronta em até 3 segundos no perfil 4G documentado. Carregar módulos pesados sob demanda, usar miniaturas autorizadas e medir custo de dependências. Não baixar vídeos completos para montar a galeria.

Memoização, virtualização e cache só com benefício demonstrado. Não assumir React Compiler habilitado. Fixar lockfile e avaliar compatibilidade, manutenção, licença e tamanho de cada biblioteca.

### Headers de segurança no documento HTML

Definir e versionar os headers servidos pelo Nginx/Amplify/CloudFront no ambiente correspondente, incluindo respostas de fallback da SPA. A política deve permitir os fluxos de captura, autenticação e mídia aprovados, sem liberar origens genéricas para resolver uma falha.

| Controle | Padrão |
| --- | --- |
| Content-Security-Policy | Fontes explícitas para scripts, estilos, imagens, fontes, mídia e conexões. Usar base-uri e object-src restritivos; inline necessário exige hash/nonce compatível com a publicação, sem adotar unsafe-inline/unsafe-eval como solução permanente. |
| Enquadramento | frame-ancestors 'none' na CSP e X-Frame-Options DENY para compatibilidade; a primeira versão não exige incorporar a aplicação em outro site. frame-ancestors deve ser enviado como header. |
| Permissions-Policy | Câmera e microfone somente para a própria origem, quando usados; negar geolocalização e capacidades desnecessárias, sem delegação genérica para iframes. A política não substitui a permissão dada pelo usuário no navegador. |
| Referrer-Policy / X-Content-Type-Options | no-referrer e nosniff, sem propagar código de evento, token ou URL privada para terceiros. |

Registrar as origens de API, S3, CloudFront e Cognito conforme sua função. Autorizar blob apenas nas diretivas de mídia/imagem em que a prévia precisar; não liberar scripts blob/data. Ajustar a política ao build estático e às respostas efetivamente servidas. Validar CSP inicialmente em Report-Only quando necessário e aplicar enforcement antes de declarar o controle atendido; sem coletar report-sample, URLs privadas ou relatórios brutos que revelem conteúdo.

As especificações [CSP](https://www.w3.org/TR/CSP3/) e [Permissions Policy](https://www.w3.org/TR/permissions-policy/) são referências técnicas em evolução; verificar suporte nas versões de navegadores testadas. Testes manuais incluem captura permitida/negada, login/retorno Cognito, upload e reprodução, além da recusa de enquadramento externo. A revisão de dependências segue os [princípios comuns](principios.md#manutenção-de-dependências).

## 11. Testes e revisão

Unitários de funções/componentes ficam no frontend e rodam automaticamente. Jornadas Playwright, compatibilidade, acessibilidade, integração e carga ficam no repositório de testes com acionamento manual.

Testar por comportamento/papel/nome acessível. Cobrir validação, transições, progresso versus confirmação, polling, retry da mesma tentativa, respostas antigas, troca de evento, bloqueio e expiração de URL. Não depender de estado interno, classes CSS ou sleeps fixos.

Na pasta `scrapbook-ui`, usar `npm ci` para instalação reproduzível, `npm run dev` para desenvolvimento e `npm run check` para lint, build com verificação de tipos e unitários. `npm test` executa unitários uma vez; `npm run test:watch` acompanha alterações. Os [comandos do frontend](https://github.com/diorgenesbackes/scrapbook-ui/blob/main/README.md#requisitos-e-execução) são a referência operacional.

- [ ] Renderização pura, estado coeso e recursos de captura liberados.
- [ ] Pastas e dependências respeitam o alcance global, da página ou da feature; `lib`, `services`, `hooks` e `utils` têm responsabilidades distintas.
- [ ] Erros, cancelamento e respostas obsoletas tratados.
- [ ] Error Boundaries e falhas de carregamento têm recuperação explícita, sem reenvio/recarga automática ou promessa de recuperar rascunho perdido.
- [ ] Headers de segurança foram validados no HTML real, incluindo fallback SPA, captura, autenticação, mídia e enquadramento.
- [ ] Limites, revisão, progresso, idempotência e confirmação seguem a API.
- [ ] Nenhum dado de recebimento aparece antes da abertura.
- [ ] Sessão/cache isolam contas e eventos; bloqueio limpa a exibição.
- [ ] Reprodução e avanço exigem ação; telas têm teclado, foco e labels.
- [ ] Build, análise estática, unitários e verificações manuais pertinentes registrados.

## Containers locais

Dockerfile multi-stage usa Node 26.13.0 para npm ci/build e Nginx unprivileged 1.30.2 para servir somente dist, com imagens fixadas por digest. Não incluir node_modules, .env ou segredos no contexto/runtime. A aplicação usa caminhos relativos; Nginx encaminha /api e /health para service:8080 na rede bridge do Compose e reconsulta o DNS após recriação. Manter query/status/headers da API e não repetir escritas. Usar upstream com pool keepalive/HTTP 1.1 e resolução DNS dinâmica, evitando conexões descartáveis por chamada. Rotas SPA usam fallback index.html; assets ausentes retornam 404. /ui-health mede Nginx, enquanto /health consulta a API.

Compose publica UI em loopback:5173, configura /tmp temporário e filesystem somente leitura, e aguarda saúde da API. Sem credenciais do banco ou URLs privadas no bundle. Scripts Vite continuam como alternativa de desenvolvimento. Publicação Amplify prevista no HLD permanece separada. [Operação local](https://github.com/diorgenesbackes/scrapbook-infrastructure/blob/feature/6/README.md); Playwright/carga continuam manuais em scrapbook-tests.
