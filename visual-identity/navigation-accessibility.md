# Navegação, responsividade e acessibilidade

[Índice](README.md) · [Matriz](screen-matrix.md)

## Shells

| Shell | Estrutura e largura | Mobile | Desktop |
| --- | --- | --- | --- |
| auth | Logo, título, formulário, recuperação/alternativa; max 416px | Uma coluna, gutter 16px; ação principal largura disponível | Centralizado; sem foto competindo com tarefa |
| account/form | Breadcrumb, título, form max 576px, status e ações | Uma coluna; grupos em sequência | Form central; label acima, nunca rótulo flutuante |
| owner | Logo, Meus eventos/Minha conta, título, ações e conteúdo; max 1200px | Nav quebra; cards uma coluna em 320px, ações empilhadas se não couberem | Cards 3 colunas quando comportarem ≥240px; informação e abertura podem usar 2 colunas |
| guest | Capa pública, título/mensagem, tarefa max 672px | Uma coluna e CTA após conteúdo; teclado não cobre ação | Coluna central com espaço lateral, sem sidebar do responsável |
| capture | Shell guest + área de prévia e controles | Área fluida; alternativa de arquivo sempre alcançável | Conteúdo central; preview mantém proporção e controles não sobrepõem informação |
| viewer | Header de evento, mídia contain, autoria/legenda, Anterior/Próximo e retorno | Controles visíveis sem gesto obrigatório | Mídia até altura disponível; texto acompanha em fluxo |
| email | Max 600px, logo, título/corpo, CTA e link alternativo | 16px padding, largura 100%, texto ≥16px | Padding 32–48px; CSS inline na implementação |
| dialog/status | Diálogo ≤480px ou estado central com ação | 16px da borda, scroll interno se necessário | Foco contido e retorno ao acionador |

Breakpoints: base 320–767px; tablet ≥48rem/768px; desktop ≥64rem/1024px. Nenhuma tela depende de largura fixa. Grid `repeat(auto-fit,minmax(min(100%,15rem),1fr))`, gap 24px, container width 100%/max-width. Não esconder campos, nomes de ações ou mensagens essenciais nas quebras. Testar 320/390/768/1440px e reflow equivalente a 400% em viewport 1280px. Tabelas podem rolar no próprio wrapper; a página não rola horizontalmente.

## Fluxos

- Conta: S01→S02/S03→S04/S08→S09; recuperação S05→S06→S02; conta S07 confirma novo e-mail e revoga sessões na troca de senha, conforme HLD. Backend inicia o fluxo gerenciado.
- Responsável: S09→S10→S11; S12 capa; S13→S14 convite; S15 compartilha QR/link para S16. Configuração de abertura salva separadamente das informações. Cancelamento retorna ao contexto anterior preservando aviso de rascunho.
- Convidado: S16→S18/S19→S20/S21/S22/S23→S25; S17 obrigatório quando identificação/concordância faltarem ou nome mudar; S24 legenda opcional; S25→S26→S27. Sucesso elimina prévia e acesso posterior do convidado.
- Acesso: S11→S28 quando bloqueado; após autorização do servidor, S29→S30. Filtros mantêm ordem primeiro→último e cursor de até dez itens. Voltar conserva filtro/posição quando ainda autorizado. A mudança para bloqueio interrompe reprodução, consultas e limpa mídia sob controle da aplicação.
- E-mails: S31→S14; S32→confirmação/recuperação; S33→autenticação e evento; S34 direciona ao runbook pelo acesso operacional habitual. Não criar painel administrativo de conteúdo fora do PRD.
- Prazos: antes da janela há data/horário e nenhuma ação de novo envio; ao meio-dia encerra novo envio; tentativa autorizada antes disso conclui até 13h. O relógio do servidor decide. Contador não autoriza ação.

## Semântica e interação

Um main por página, hierarquia h1→h2, header/nav/footer/section com nome, label explícito e fieldset/legend por grupo. Todas as funções essenciais devem ser operáveis com teclado e toque. Alvos adotados ≥44×44px, preferencialmente 48px, espaço ≥8px. Link navega; botão executa; ícone isolado exige nome acessível.

Menu por disclosure mantém tabulação normal; se usar role=menu no futuro, implementar o padrão completo de setas/retorno. Abas usam setas/Home/End, seleção sem foco perdido e painel associado. Dialog nativo: foco inicial na ação segura, Tab contido, Escape cancela, retorno ao acionador. Não esconder o foco atrás de barras fixas; respeitar scroll-margin e safe-area em controles fixos, se adotados.

Body 16px/1.6, zoom sem bloqueio, conteúdo quebra palavras longas e suporta texto 200%/reflow 400%. Estado inválido associa mensagem e preserva o valor. Loading e sucesso usam status; erro urgente usa alert sem anúncios repetidos. Reduzir movimento, permitir forced-colors e não comunicar apenas por cor.

## Proteção da surpresa e da pessoa

Bloqueio é uma composição própria, sem quantidade, remetentes, mídia, skeleton de fotos, requests de galeria ou URLs utilizáveis no DOM. Fotos do catálogo são sintéticas; a implementação não deve usar pré-carregamento de conteúdo bloqueado. Renderizar conteúdo de usuário como texto seguro. Confirmação de convidado mostra apenas recibo genérico e próximas ações, sem prévia do que foi enviado.

Ícones/foco/texto foram especificados para [WCAG 2.2 AA](https://www.w3.org/TR/WCAG22/); a validação completa exige leitura assistiva, dispositivos reais, fluxo de captura e revisão de mídia acessível/menores (Q03/Q05). O cálculo de contraste e testes de catálogo não substituem essas avaliações. Não há alegação de conformidade integral nesta entrega.
