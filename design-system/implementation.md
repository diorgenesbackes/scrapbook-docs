# Aplicação do sistema de design

[Catálogo](index.html) · [Matriz](screen-matrix.md) · [Tokens](tokens.css) · [Componentes](components.md)

A identidade visual fornece marca, fotografia, paleta, tipografia e os exemplos iniciais. O PRD define os fluxos e critérios de produto. O HLD detalha dependências e revisões explícitas, como confirmação de e-mail, recuperação por código e autorização de galeria. O catálogo local é uma proposta de design e usa dados fictícios.

## Composição

| Área | Composição compartilhada | Decisão de UX |
| --- | --- | --- |
| Conta e autenticação | Formulário estreito, labels persistentes, confirmação ou recuperação | Priorizar o próximo passo no Cognito; manter colagem, preenchimento automático e gerenciadores de senha. |
| Organização | Cabeçalho, lista de eventos, formulário, definição de abertura e compartilhamento | Mostrar o que ainda pode ser editado. Preservar dados digitados em conflitos e consultar o resultado se o salvamento ficar incerto. |
| Participação | Capa pública, identificação, escolha de formato, área de captura e revisão | Permitir participação sem conta, oferecer seleção de arquivo quando a captura falhar e pedir confirmação antes do envio. |
| Recebimento | Progresso de bytes, validação, recuperação e confirmação | Separar transferência e persistência. Ao perder resposta, consultar o mesmo envio. |
| Lembranças | Tela de espera, galeria filtrável e visualizador | Consultar autorização antes de buscar mídia ou metadados. Mostrar bloqueio separado de vazio. Reproduzir e avançar por ação explícita. |
| E-mails e erros | Assunto informativo, texto curto, ação contextual e referência de suporte | Manter compreensão com imagens desativadas. Exibir correlação somente quando fornecida. |

## Estados que mudam a decisão do usuário

- **Prazo.** Exibir horário de Brasília por extenso quando houver risco de ambiguidade. A janela abre duas horas antes do evento e fecha ao meio-dia do dia seguinte. Até as 13h, apenas tentativas já iniciadas no prazo podem concluir a transferência. O servidor decide elegibilidade.
- **Envio.** Uma foto em depoimento ou até dez fotos em momento, com 20 MB por foto. Vídeo de até 500 MB, com cinco minutos em depoimento ou um minuto em momento. Áudio de depoimento de até 50 MB e cinco minutos. Texto até 5.000 caracteres e legenda opcional até 500. Um conjunto de fotos recebe confirmação integral.
- **Recuperação.** Manter o rascunho em memória durante erro recuperável. Reutilizar a tentativa lógica ao repetir ou consultar um envio. Um refresh perde o rascunho. Exigir nova concordância ao mudar o nome.
- **Abertura.** Manter o bloqueio até a confirmação do servidor. Na troca para abertura futura, remover acesso sob controle da aplicação e cancelar carregamentos ativos. URLs já emitidas podem continuar válidas por até dois minutos. O e-mail avisa sobre a abertura, que é autorizada independentemente.
- **Edição.** O primeiro recebimento confirmado aciona o travamento de dados e convites. O HLD aceita consistência eventual com meta de até cinco segundos em condições normais. A interface deve tratar rejeições e atualizar os controles segundo o servidor.
- **Autenticação.** Confirmação de e-mail no cadastro e antes de trocar o endereço. Recuperação por código Cognito de uma hora. Validar na experiência hospedada os textos, os estados, preenchimento automático e operação assistiva. O catálogo de marca representa a intenção visual.

## Verificação de acessibilidade

Use as especificações de [navegação e acessibilidade](navigation-accessibility.md) junto à referência [WCAG 2.2](https://www.w3.org/TR/WCAG22/).

| Verificação | Resultado esperado |
| --- | --- |
| Teclado e foco | Link para conteúdo, ordem coerente, foco sempre visível, setas nas abas, Escape em diálogos e foco devolvido quando a pessoa permanece na tela. |
| Refluxo e toque | Testar páginas a 320 CSS px, com zoom e espaçamento de texto ampliado. Controles principais têm 48 px, com meta de 44 px para alvos do produto. |
| Contraste e estados | Texto comum com pelo menos 4,5:1. Componentes visuais e texto grande com pelo menos 3:1 onde exigido. Combinar símbolo ou cor com mensagem explícita. |
| Erros e mudanças | Vincular erro ao campo, preservar dados, anunciar confirmações e transições importantes. Evitar leitura de cada atualização percentual e de cada minuto do contador. |
| Imagens e mídia | Descrever imagens informativas, usar texto alternativo vazio em imagens redundantes e deixar reprodução ao comando da pessoa. Validar equivalentes textuais, legendas e audiodescrição aplicáveis a cada tipo de conteúdo; uma legenda livre do convidado tem finalidade diferente. |
| QR e e-mail | Fornecer endereço em texto e ação acessível equivalente ao QR. Confirmar leitura de e-mail com imagens desativadas, sem depender de cor, animação ou imagem de código. |
| Validação humana | Percorrer os fluxos com VoiceOver/Safari e NVDA com navegador compatível, além de teclado, toque e aparelhos reais. Cobrir Cognito, câmera, microfone, falhas de rede e clientes de e-mail no ambiente integrado. |

## Uso do catálogo

Em **Telas**, busque por nome ou ID, escolha o estado e alterne a largura da prévia. Para experimentar o fluxo de texto, abra **Página dos convidados → Deixe um depoimento → identificação e concordância → Texto → Revisar → Enviar**. Na transferência, simule perda de conexão, retome o envio, conclua a transferência e confirme o recebimento. Use apenas nomes e textos fictícios. A demonstração mantém esses valores somente na memória da página.

Os exemplos de captura, QR, serviço de e-mail, autenticação e reprodução usam referências estáticas ou controles simulados. Para cada página de produção, use a especificação correspondente da matriz e valide o fluxo com os dados e os serviços reais.
