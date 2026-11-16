# Componentes e estados

[Índice](README.md) · [CSS](components.css) · [Prévia interativa](index.html#components)

## Contrato comum

Controles herdam tokens; default/hover/focus/active são estados de interação, não rotas. Hover só complementa informação. Focus usa :focus-visible (3px/3px), active só durante pressionamento; disabled usa atributo nativo e motivo textual. Loading usa aria-busy, texto no presente (“Salvando…”), trava acionamento e mantém nome/contexto. Selected usa semântica apropriada, não só cor. Error associa mensagem com aria-describedby/aria-invalid. Sucesso breve usa role=status; erro urgente, role=alert, uma vez por operação.

| Componente / classe | Dimensão e variantes | Estados e uso |
| --- | --- | --- |
| Botão `.sb-button` | Altura 48px, padding 10×20px, radius 8px; primary, secondary, quiet, danger, icon | Default/hover/focus/active/disabled/loading. Primary único por etapa; danger apenas descarte local/ação com consequência, com confirmação. selected não se aplica: usar tabs/radio/aria-pressed para escolha. |
| Campo `.sb-input` | 48px, padding 11×14px, borda 1px; input/email/password/date/file/textarea | Default/hover/focus/disabled/error. Ativo é edição; loading da operação no form. Valor válido preservado após erro, mensagem por campo. Placeholder nunca substitui label. |
| Select `.sb-input` | Mesma caixa do input; opções nativas | Default/hover/focus/disabled/selected/error; loading bloqueia opções dependentes. Não reinventar popup sem necessidade. |
| Checkbox `.sb-check` | Caixa 20px, linha-alvo ≥44px, gap 12px | Hover/focus/active/checked/disabled/error. Label toda clicável. Concordância required; nome alterado limpa o check e exige nova concordância. |
| Radio `.sb-check` | Mesmo alvo; fieldset/legend por grupo | Default/hover/focus/active/selected/disabled/error. Setas mudam escolha nativa; uma escolha por grupo, sem selecionar opções conflitantes. |
| Card `.sb-card` | Radius 12px, borda decorativa 1px, conteúdo 24px; mídia 4:3 | Default; card-link adiciona hover/focus/active. Selected só em escolha explícita (radio), nunca usar card inteiro com botões aninhados. Loading por skeleton de dimensões estáveis; erro/processing com texto; indisponível não oferece link inútil. |
| Tabela `.sb-table` | Células 14px/1.6, padding 14px, th com scope | Caption identifica o conjunto; cabeçalhos persistem em responsivo. Wrapper rolável é focável/nomeado se necessário; colunas de dados não desaparecem. Loading/error/empty como estado do conjunto. Não inventar tabela de recebimentos bloqueados. |
| Alerta `.sb-alert` | Padding 16px, gap 12px, ícone 20px, radius 8px | Info/success/warning/error; texto explica ação. Borda/texto semânticos; loading por status. Sem hover/active/selected em alerta estático; links/botões internos herdam o contrato comum. |
| Diálogo `.sb-dialog` | Max 480px/calc(100vw−32px), padding 32px, backdrop 60% | Abrir por ação explícita; foco inicial na alternativa segura; Tab contido; Escape cancela; fechar retorna foco. Pendente trava confirmação; erro mantém aberto e informa. Nunca auto-confirmar. |
| Abas `.sb-tabs/.sb-tab` | Alvo ≥44px, padding 8×16px, gap 8px | Hover/focus/active/selected/disabled/loading/error do painel. role=tablist/tab/tabpanel, aria-selected/controls, roving tabindex; setas/Home/End. Clique não dispara envio ou autoplay. |
| Breadcrumb `.sb-breadcrumbs` | 14px, separador decorativo, quebra de linha | Links focáveis; atual aria-current=page. Hover/focus/active dos links. Não é stepper nem estado de upload. |
| Navegação | Logo/links, itens ≥44px; wrap no mobile | aria-label por região e aria-current no destino. Mobile usa disclosure, menu acionado por teclado/toque. Sem esconder ações essenciais só em hover. |
| Badge `.sb-badge` | 12px, padding 4×10px, radius pill | Rótulo estático legível; não imitar botão ou expor contagens durante bloqueio. Selected pertence ao controle que seleciona. |
| Progresso `.sb-progress` | Barra 8px, largura 100% | Indeterminado na autorização/validação; valor real max=100 na transferência; erro preserva conteúdo; concluído só após persistência confirmada. 100% de bytes não é sucesso. |
| Estado vazio `.sb-empty` | Padding clamp(32px,6vw,80px), texto e ação | Vazio autorizado, erro, indisponível e bloqueado são composições diferentes. Não mostrar skeleton de fotos nem número de itens em bloqueio. |
| Player | Foto contain; vídeo/áudio com controles nativos acessíveis | Play/pause/volume/posição/erro/processing/URL expirada. Sem autoplay, avanço automático, download ou exclusão. Fim da mídia mantém o item atual. |

## Validação e microcopy

Label curto, sentença simples, verbo de ação: “Criar evento”, “Salvar informações”, “Salvar abertura”, “Enviar”, “Tentar novamente”, “Continuar”. Evitar “OK”, “Erro 500” e jargão em tarefas de convidados. Datas e limites aparecem antes da ação. “Informações indisponíveis para edição” não revela o motivo do travamento.

Campos obrigatórios usam required e indicação textual consistente no formulário. O primeiro envio inválido foca o primeiro campo inválido; resumo de erros com links quando houver mais de um. Erros de rede não apagam valores. Sucesso mantém foco previsível; mudança de tela foca título/main. Não anunciar cada porcentagem por live region: agrupar marcos e anunciar troca de etapa.

Mensagem obrigatória de saída: “Você pode perder o trabalho em andamento.” Botões “Permanecer” / “Sair e descartar”. Mensagem de prazo: “O prazo para concluir este envio terminou.” Erro de campo explica o limite (“Use até 500 caracteres na legenda.”). Falha com resposta perdida consulta a mesma tentativa antes de repetir escrita.

## Ícones e conteúdo

Ícones complementam texto, 20px, traço 1.7 no viewBox 24. A origem é o próprio catálogo (sem pacote externo); [ativos/direitos](assets/README.md). Fotos com alt contextual após autorização; capa decorativa usa alt vazio quando título adjacente já comunica tudo. Texto de convidado é conteúdo não confiável: renderizar como texto, nunca HTML.
