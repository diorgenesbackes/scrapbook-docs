# Guideline de frontend

[Índice](README.md) · [Princípios](principios.md) · [API](api.md) · [Mensageria e mídias](mensageria.md)

Aplicação: React e TypeScript, frontend estático previsto no Amplify, prioritariamente móvel e em português do Brasil. React 19.3.1, React Router 8.5.1, TypeScript 7.0.2, Vite 8.3.6, Oxlint 1.90.0 e Vitest 5.1.0/Testing Library/jsdom estão fixados. Node 26.13.0/npm 11.20.0 são as versões validadas. Consulte [execução e configuração na feature](https://github.com/diorgenesbackes/scrapbook-ui/blob/feature/GE-STORY-001/README.md) e lockfile. Playwright 1.65.0 é manual em scrapbook-tests; publicação ainda não configurada.

## 1. Organização por funcionalidade

Estrutura proposta, criando apenas pastas com conteúdo:

```text
src/
  app/                       # inicialização, rotas e providers
  features/
    identity/                # sessão, conta e redirecionamentos
    events/                  # configuração, responsáveis e convites
    contributions/           # identificação, captura, revisão e envio
    gallery/                 # bloqueio, filtros, sequência e reprodução
  shared/
    api/                     # transporte e interpretação de erros
    ui/                      # controles reutilizados
    lib/                     # funções pequenas com finalidade clara
```

Contratos, componentes, hooks e unitários ficam próximos da funcionalidade. `shared` não depende de `features`; colaboração entre features usa interfaces pequenas, sem importar detalhes internos ou criar ciclos por reexports.

Componentes não conhecem schema SQL ou SDK de backend. O adaptador converte respostas nos modelos da tela, sem reproduzir as quatro camadas .NET no React.

## 2. Componentes, composição e hooks

- Renderização pura; não gravar, iniciar upload ou alterar props durante render.
- Estado atualizado por novas referências; composição preferida a herança.
- Hooks chamados segundo suas regras, com cleanup de subscriptions, timers e captura.
- Chaves estáveis em listas; não usar índice para itens reordenáveis nem gerar UUID a cada render.
- Extrair componente/hook quando houver responsabilidade ou reutilização real.
- Não tornar uma função pura em hook sem necessidade.

Captura, revisão, upload e galeria são interações distintas. Uma legenda compartilhada por um conjunto não exige manter cópias editáveis independentes em cada foto.

## 3. Estado e ciclo de vida

| Informação | Local e cuidados |
| --- | --- |
| Campo, diálogo, etapa de captura | Estado local ou reducer da jornada. |
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

Centralizar base URL, headers, credenciais, cancelamento e interpretação de erros numa camada pequena. Contratos específicos pertencem à funcionalidade.

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

Validar Android, iPhone e desktop nas duas versões principais mais recentes dos navegadores previstos no PRD, registrando versões efetivamente testadas. Testar permissão negada, captura indisponível, orientação, conteúdo longo e rede instável. Incluir pessoas de diferentes idades nos testes do protótipo.

## 10. Segurança, desempenho e dependências

Não renderizar texto não confiável como HTML. Não registrar depoimento, legenda, nome, blob, tokens ou URLs utilizáveis em telemetria. Link/QR code público permite participação no evento, não leitura das lembranças.

Configuração incluída no bundle é pública. Se Vite for adotado, `VITE_*` nunca contém segredos. Variável de runtime não altera automaticamente arquivo estático já compilado.

Priorizar a página dos convidados pronta em até 3 segundos no perfil 4G documentado. Carregar módulos pesados sob demanda, usar miniaturas autorizadas e medir custo de dependências. Não baixar vídeos completos para montar a galeria.

Memoização, virtualização e cache só com benefício demonstrado. Não assumir React Compiler habilitado. Fixar lockfile e avaliar compatibilidade, manutenção, licença e tamanho de cada biblioteca.

## 11. Testes e revisão

Unitários de funções/componentes ficam no frontend e rodam automaticamente. Jornadas Playwright, compatibilidade, acessibilidade, integração e carga ficam no repositório de testes com acionamento manual.

Testar por comportamento/papel/nome acessível. Cobrir validação, transições, progresso versus confirmação, polling, retry da mesma tentativa, respostas antigas, troca de evento, bloqueio e expiração de URL. Não depender de estado interno, classes CSS ou sleeps fixos.

Na pasta `scrapbook-ui`, usar `npm ci` para instalação reproduzível, `npm run dev` para desenvolvimento e `npm run check` para lint, build com verificação de tipos e unitários. `npm test` executa unitários uma vez; `npm run test:watch` acompanha alterações. Os [comandos do frontend](https://github.com/diorgenesbackes/scrapbook-ui/blob/main/README.md#requisitos-e-execução) são a referência operacional.

- [ ] Renderização pura, estado coeso e recursos de captura liberados.
- [ ] Erros, cancelamento e respostas obsoletas tratados.
- [ ] Limites, revisão, progresso, idempotência e confirmação seguem a API.
- [ ] Nenhum dado de recebimento aparece antes da abertura.
- [ ] Sessão/cache isolam contas e eventos; bloqueio limpa a exibição.
- [ ] Reprodução e avanço exigem ação; telas têm teclado, foco e labels.
- [ ] Build, análise estática, unitários e verificações manuais pertinentes registrados.
