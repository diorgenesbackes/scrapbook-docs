### PRD: Scrap Book Primeira versão da plataforma de lembranças de eventos

Versão: 1.0
Data: 2026-11-10
Responsável: Diórgenes Backes

---

### Resumo

O Scrap Book será uma plataforma web para reunir depoimentos e momentos registrados pelos convidados de um evento. Seu propósito é preservar mensagens pessoais, emoções e registros espontâneos que hoje ficam dispersos em celulares, Stories, postagens ou recados de papel.

A primeira utilização será no casamento de Diórgenes Backes. A plataforma também permitirá cadastro público e criação de eventos por outras pessoas, em homologação e produção, incluindo aniversários e encontros entre amigos e familiares.

Os convidados participarão sem login, por link ou QR code, enviando fotos, vídeos, áudios ou textos conforme a categoria escolhida. Os responsáveis pelo evento poderão acessar os registros imediatamente ou programar sua abertura futura.

A experiência será prioritariamente móvel, com funcionamento adequado em computadores e acessibilidade para pessoas de diferentes idades e familiaridade com tecnologia.

A entrega deverá estar em produção e validada até **15/10/2027**, seguindo a sequência: PRD, HLD, guidelines, desenvolvimento, testes locais, homologação, testes em homologação, implantação e testes em produção.

---

### Contexto e problema

Público-alvo

- Responsáveis por casamentos, aniversários e encontros entre amigos e familiares.
- Convidados com idade aproximada entre 10 e 90 anos.
- Pessoas com diferentes níveis de familiaridade com celulares e computadores.
- Participantes externos que testarão seus próprios eventos em homologação.

Cenários de uso chave

- Convidado registra uma mensagem pessoal para os responsáveis.
- Convidado captura momentos espontâneos e envia várias contribuições durante o evento.
- Responsáveis acompanham registros no modo imediato.
- Responsáveis guardam registros para abertura em uma data futura.
- Responsáveis revisitam as lembranças em galeria ou sequência de visualização.
- Uma pessoa cria ou participa de vários eventos independentes.

Onde essa feature será implantada

- Novo sistema web, chamado Scrap Book.
- Ambientes local, homologação e produção.
- Infraestrutura AWS em Norte da Virgínia, região `us-east-1`.
- Homologação e produção na mesma conta AWS, com recursos, dados e permissões isolados.

Problemas priorizados

- **Alta prioridade:** fotos e vídeos espontâneos ficam dispersos em celulares e redes sociais, dificultando sua recuperação e preservação.
- **Alta prioridade:** recados escritos não preservam voz, gestos e entonação, limitando a transmissão da emoção original.
- **Alta prioridade:** os responsáveis não conseguem acompanhar todos os momentos do evento e perdem cenas registradas pelos convidados.
- **Alta prioridade:** uma experiência complicada pode impedir a participação de convidados com pouca familiaridade tecnológica.
- **Alta prioridade:** uma cápsula do tempo perde seu propósito se os registros puderem ser acessados antes da abertura programada.

---

### Objetivos e métricas

| Objetivo | Métrica | Meta |
| ---------------------------------------------------------------------- | --------------------------------------------------------------- | ------------------------- |
| Estimular participação no casamento piloto | Proporção estimada de convidados presentes que enviaram ao menos um registro | Pelo menos 70%; aproximadamente 63 a 70 pessoas para 90 a 100 presentes |
| Facilitar o uso sem assistência | Proporção de participantes do teste que concluem um envio sem ajuda | Meta numérica pendente dos testes do protótipo |
| Preservar envios confirmados | Quantidade de registros confirmados perdidos nos cenários de falha cobertos | Zero |
| Preservar a abertura programada | Acessos indevidos a conteúdo ou metadados bloqueados nos testes | Zero |
| Atender a carga inicial | Envios simultâneos suportados em dois eventos | 200, considerando 100 convidados por evento |
| Disponibilizar mídias para reprodução | Tempo entre confirmação do envio e mídia pronta | Pelo menos 95% em até 5 minutos |
| Garantir qualidade de lançamento | Critérios de aceitação aprovados e defeitos críticos ou de alta gravidade em aberto | 100% dos critérios aprovados e nenhum desses defeitos aberto |

A participação será estimada. Nome ou apelido não comprova identidade, e a plataforma não terá vínculo com lista oficial de convidados.

---

### Escopo

Incluso

- Cadastro público com nome, e-mail e senha.
- Contas individuais e edição dos dados da conta.
- Login, encerramento de sessão e recuperação de senha.
- Criação e participação em vários eventos por conta.
- Até dois responsáveis com permissões iguais por evento.
- Convite do segundo responsável por QR code ou e-mail.
- Evento utilizável com apenas um responsável.
- Nome próprio para cada evento.
- Capa e mensagem padrão, com personalização opcional.
- Página pública para convidados, acessível por link e QR code.
- QR code disponível para impressão.
- Depoimentos em foto, vídeo, áudio ou texto.
- Momentos em foto ou vídeo.
- Captura pela página e seleção de arquivos existentes.
- Revisão antes do envio.
- Nome ou apelido obrigatório, reutilizado no navegador.
- Concordância com o aviso de acesso ao conteúdo.
- Legendas opcionais para mídias.
- Janela de envio vinculada ao horário do evento.
- Progresso do envio, prevenção de duplicações e novas tentativas.
- Acesso imediato ou abertura futura.
- Galeria e sequência de visualização.
- Filtros Depoimentos, Momentos e Tudo.
- Armazenamento por tempo indeterminado enquanto a plataforma for mantida.
- E-mails de convite, recuperação de senha e liberação.
- Monitoramento, auditoria, alertas e recuperação de falhas.
- Mesmas regras funcionais para eventos reais e eventos de teste.

Fora de escopo

- Aplicativo nativo.
- Funcionamento offline e preservação de rascunhos após sair ou recarregar a página.
- Download de lembranças pela interface.
- Edição ou exclusão de registros após o envio.
- Exclusão de eventos pela interface.
- Administradores adicionais, como madrinha ou organizador.
- Mais de dois responsáveis por evento.
- Login ou cadastro obrigatório de convidados.
- Vínculo com lista oficial de convidados.
- Busca de registros por nome ou apelido.
- Edição, corte ou montagem de mídias.
- Confirmação inicial do e-mail de cadastro.
- Autenticação em duas etapas.
- CAPTCHA no uso normal.
- Reprodução ou avanço automáticos.
- Interface ou e-mails em outros idiomas.
- Escolha de fuso horário.
- Extensão automática da janela de envio por indisponibilidade.
- Recuperação entre regiões AWS.
- Proteção contra um administrador que deliberadamente altere permissões para contornar o bloqueio.
- Funcionalidades comerciais, como planos e cobrança, nesta versão.

---

### Requisitos funcionais

#### FR-001 Cadastro, autenticação e sessão

Permitir cadastro público e autenticação individual dos responsáveis.

**Fluxo principal**

- Solicitar nome, e-mail e senha.
- Exigir senha com pelo menos 6 caracteres, incluindo uma letra maiúscula e uma minúscula.
- Permitir mostrar ou ocultar a senha.
- Permitir login com e-mail e senha, sem nome de usuário separado.
- Manter a sessão por até 7 dias.
- Permitir sair da conta.

**Fluxos alternativos e exceções**

- O cadastro não exigirá confirmação inicial de e-mail.
- Uma conta poderá criar eventos e aceitar convites para vários eventos.

**Erros previstos**

- Dados obrigatórios ausentes ou inválidos.
- E-mail já associado a uma conta.
- Senha fora da regra estabelecida.
- Credenciais inválidas ou sessão expirada.
- Excesso de tentativas de login.

**Prioridade:** alta

---

#### FR-002 Edição da conta e recuperação de senha

Permitir alterar nome, e-mail e senha, além de recuperar o acesso por e-mail.

**Fluxo principal**

- Permitir editar os dados da própria conta.
- Exigir a senha atual para alterar e-mail ou senha.
- Na recuperação, enviar um link pelo Amazon SES ao e-mail cadastrado.
- Aceitar o link por 30 minutos e uma única vez.
- Encerrar todas as sessões após troca ou recuperação de senha.

**Fluxos alternativos e exceções**

- Quem esqueceu a senha utilizará a recuperação, sem informar a senha atual.
- Alterar o nome não exigirá a senha atual.

**Erros previstos**

- Senha atual incorreta.
- Novo e-mail já utilizado.
- Link inválido, expirado ou já utilizado.
- Nova senha fora da regra.
- Falha no envio do e-mail.

**Prioridade:** alta

---

#### FR-003 Criação e gestão de múltiplos eventos

Permitir que uma conta crie e participe de vários eventos independentes.

**Fluxo principal**

- Informar nome, data e horário do evento.
- Aplicar exclusivamente o horário de Brasília.
- Disponibilizar capa e mensagem de boas-vindas padrão.
- Permitir personalizar capa e mensagem antes do primeiro registro.
- Configurar o modo de acesso.
- Exibir ao responsável somente os eventos aos quais está vinculado.

**Fluxos alternativos e exceções**

- O evento poderá funcionar com apenas um responsável.
- Eventos de teste seguirão as mesmas regras dos eventos reais.
- Não haverá exclusão de eventos nesta versão.

**Erros previstos**

- Dados obrigatórios ausentes ou inválidos.
- Tentativa de acessar ou configurar evento sem vínculo.
- Tentativa de modificar dados já travados.

**Prioridade:** alta

---

#### FR-004 Convite do segundo responsável

Permitir vincular um segundo responsável com as mesmas permissões do criador.

**Fluxo principal**

- Disponibilizar uma página de convite com QR code apresentado imediatamente.
- Oferecer envio do convite por e-mail.
- Permitir ao destinatário criar uma conta ou entrar em uma existente.
- Vincular a conta ao evento após aceitar o convite.
- Invalidar o convite após seu uso.

**Fluxos alternativos e exceções**

- O convite terá validade de 24 horas.
- Gerar um novo convite invalidará qualquer convite anterior não utilizado.
- Uma conta poderá aceitar convites para vários eventos.
- O primeiro registro recebido bloqueará a criação e a aceitação de convites.
- Não será exibido aviso específico recomendando vincular o segundo responsável antes do primeiro registro.

**Erros previstos**

- Convite inválido, expirado, substituído ou utilizado.
- Evento com dois responsáveis.
- Convite aceito após o primeiro registro.
- Falha no envio do e-mail.

**Prioridade:** alta

---

#### FR-005 Página dos convidados e QR code

Disponibilizar uma página pública por evento para participação sem login.

**Fluxo principal**

- Permitir acesso pelo link ou QR code, sem senha adicional.
- Exibir nome, capa e mensagem do evento.
- Apresentar as opções Deixe um depoimento e Registre um momento.
- Exibir o QR code na página dos convidados.
- Disponibilizar aos responsáveis uma versão do QR code para baixar e imprimir.

**Fluxos alternativos e exceções**

- Antes da abertura, informar quando os envios estarão disponíveis.
- Após o encerramento, informar que novos envios não são aceitos.
- A página pública não permitirá visualizar registros enviados.

**Erros previstos**

- Evento inexistente ou link inválido.
- Falha no carregamento da página ou geração do QR code.

**Prioridade:** alta

---

#### FR-006 Janela de envio e tolerância para conclusão

Controlar o recebimento conforme data e horário do evento.

**Fluxo principal**

- Abrir os envios duas horas antes do início do evento.
- Encerrar novos envios ao meio-dia do dia seguinte.
- Considerar como início o acionamento de Enviar após a revisão.
- Permitir concluir ou repetir envios iniciados dentro do prazo até as 13h do dia seguinte.
- Aplicar todos os horários em Brasília.

**Fluxos alternativos e exceções**

- Começar uma gravação ou selecionar um arquivo não contará como início do envio.
- Arquivos recebidos integralmente no prazo poderão ser processados após as 13h.
- Indisponibilidade não estenderá automaticamente os prazos.
- A retomada dependerá da preservação do conteúdo na página.

**Erros previstos**

- Tentativa de iniciar antes da abertura ou após o encerramento.
- Tentativa de concluir ou repetir após a tolerância.
- Exibir após o prazo: “O prazo para concluir este envio terminou.”

**Prioridade:** alta

---

#### FR-007 Identificação e concordância do convidado

Solicitar nome ou apelido e concordância com o aviso de acesso ao conteúdo.

**Fluxo principal**

- Exigir nome ou apelido em Depoimentos e Momentos.
- Informar que os registros serão acessíveis somente aos responsáveis pelo evento.
- Solicitar concordância antes do envio.
- Salvar nome e concordância no navegador, vinculados ao evento.
- Reutilizar esses dados nos próximos envios das duas categorias.

**Fluxos alternativos e exceções**

- Permitir alterar o nome antes de cada envio.
- Solicitar nova concordância quando o nome for alterado.
- Não validar o nome contra cadastro ou lista oficial.
- Caso os dados locais sejam removidos, solicitar o preenchimento novamente.

**Erros previstos**

- Nome vazio.
- Concordância não fornecida.
- Tentativa de envio sem os dados exigidos.

**Prioridade:** alta

---

#### FR-008 Depoimentos

Permitir mensagens pessoais em um formato por envio.

**Fluxo principal**

- Oferecer foto, vídeo, áudio e texto.
- Permitir escolher um formato.
- Capturar ou preencher o conteúdo.
- Revisar e enviar.

**Fluxos alternativos e exceções**

- Permitir vários depoimentos da mesma pessoa.
- Permitir formatos diferentes em envios separados.
- Para foto, aceitar uma por envio.
- Não combinar formatos em um mesmo depoimento.

**Erros previstos**

- Conteúdo ausente ou vazio.
- Mais de uma foto no mesmo depoimento.
- Conteúdo acima dos limites definidos.

**Prioridade:** alta

---

#### FR-009 Momentos

Permitir registrar cenas do evento em fotos ou vídeos.

**Fluxo principal**

- Oferecer envio de até 10 fotos por vez ou um vídeo por envio.
- Permitir capturar ou selecionar os arquivos.
- Revisar e enviar.
- Permitir novos registros após a conclusão.

**Fluxos alternativos e exceções**

- Um conjunto de fotos será tratado como um envio indivisível.
- Se uma foto falhar, nenhuma foto do conjunto ficará registrada como envio concluído.
- Após sucesso, cada foto aparecerá separadamente na galeria, com o mesmo nome e legenda.

**Erros previstos**

- Mais de 10 fotos.
- Mais de um vídeo.
- Falha em qualquer foto do conjunto.
- Arquivo vazio, inválido ou acima dos limites.

**Prioridade:** alta

---

#### FR-010 Captura e seleção de arquivos

Permitir capturar conteúdo pela página ou usar arquivos existentes.

**Fluxo principal**

- Oferecer câmera para fotos e vídeos.
- Oferecer microfone para áudios.
- Oferecer seleção de arquivos do dispositivo.
- Mostrar tempo restante durante gravações.
- Encerrar a gravação automaticamente no limite e levar à revisão.

**Fluxos alternativos e exceções**

- Não enviar automaticamente ao terminar a gravação.
- Se a permissão for negada, orientar como liberá-la e oferecer seleção de arquivo.
- Se a captura não estiver disponível, preservar a alternativa de seleção de arquivo.

**Erros previstos**

- Permissão negada.
- Dispositivo de captura indisponível.
- Interrupção da gravação.
- Falha ao acessar um arquivo.

**Prioridade:** alta

---

#### FR-011 Limites e validação de conteúdo

Validar tamanho, duração e conteúdo antes de confirmar o recebimento.

**Fluxo principal**

- Fotos: até 20 MB cada.
- Vídeos: até 500 MB por arquivo.
- Vídeos de Depoimentos: até 5 minutos.
- Vídeos de Momentos: até 1 minuto.
- Áudios: até 50 MB e 5 minutos.
- Textos: até 5.000 caracteres, incluindo espaços e pontuação.
- Legendas: até 500 caracteres.

**Fluxos alternativos e exceções**

- Não estabelecer duração mínima para gravações, mas rejeitar arquivos vazios.
- Não oferecer corte ou edição para adequar arquivos.
- A lista de formatos e codecs aceitos será definida no HLD e validada nos dispositivos previstos.

**Erros previstos**

- Tamanho ou duração excedidos.
- Arquivo vazio, corrompido ou de formato não suportado.
- Texto obrigatório vazio.
- Mensagem de erro deverá explicar o motivo e o limite aplicável.

**Prioridade:** alta

---

#### FR-012 Texto e legendas opcionais

Permitir mensagens escritas e legendas sem sugerir que o convidado precisa produzir um texto longo.

**Fluxo principal**

- No depoimento em texto, ocultar inicialmente o contador.
- Mostrar aviso quando restarem 500 caracteres, atualizando o saldo.
- Nas mídias, apresentar “Deseja adicionar uma legenda?”.
- Abrir o campo de legenda somente após essa escolha.
- Na legenda, mostrar aviso quando restarem 100 caracteres.
- Impedir novos caracteres ao atingir o limite, preservando edição e exclusão.

**Fluxos alternativos e exceções**

- A legenda será opcional.
- Em conjunto de fotos, uma legenda será aplicada a todas as imagens.
- A legenda não constituirá um segundo formato de depoimento.

**Erros previstos**

- Tentativa de ultrapassar o limite por digitação ou colagem.
- Conteúdo acima do limite recebido pelo backend.

**Prioridade:** alta

---

#### FR-013 Revisão e proteção do trabalho em andamento

Permitir revisar e descartar o conteúdo antes do envio.

**Fluxo principal**

- Mostrar o conteúdo antes de enviá-lo.
- Permitir refazer gravações, trocar fotos, editar textos e descartar o registro.
- Solicitar ação explícita em Enviar.
- Avisar sobre perda do trabalho ao tentar sair com conteúdo pendente.

**Fluxos alternativos e exceções**

- Nas ações controladas pela aplicação, usar “Você pode perder o trabalho em andamento.”
- Ao fechar ou recarregar, o navegador poderá apresentar seu próprio aviso.
- Não garantir preservação após sair ou recarregar a página.

**Erros previstos**

- Falha na prévia.
- Perda do conteúdo local após saída ou recarga.
- Interrupção do navegador.

**Prioridade:** alta

---

#### FR-014 Envio, progresso e novas tentativas

Receber o conteúdo sem duplicações e confirmar somente após armazenamento completo.

**Fluxo principal**

- Solicitar autorização temporária ao backend.
- Enviar mídias diretamente ao S3.
- Exibir progresso em porcentagem.
- Desabilitar novos acionamentos de Enviar.
- Impedir iniciar outro registro enquanto houver envio em andamento.
- Confirmar sucesso após todos os arquivos e dados necessários estarem armazenados.

**Fluxos alternativos e exceções**

- Em falha, manter o conteúdo na página para nova tentativa.
- Aplicar a tolerância de prazo somente ao envio iniciado dentro da janela.
- Não registrar duplicatas causadas por novas tentativas.
- Em conjuntos de fotos, aplicar sucesso ou falha ao conjunto inteiro.
- Arquivos parciais não serão apresentados como lembranças recebidas.

**Erros previstos**

- Perda de conexão.
- Falha de armazenamento ou persistência.
- Autorização inválida ou expirada.
- Prazo encerrado.
- Confirmação não recebida pelo navegador.

**Prioridade:** alta

---

#### FR-015 Confirmação e restrições do convidado

Confirmar o recebimento sem permitir acesso posterior ao conteúdo enviado.

**Fluxo principal**

- Mostrar confirmação após sucesso.
- Oferecer Deixar outro depoimento e Registrar um momento enquanto novos envios estiverem permitidos.
- Impedir visualização ou exclusão do registro pelo convidado após o envio.

**Fluxos alternativos e exceções**

- Se a janela já estiver encerrada, não permitir iniciar outro registro.
- A revisão anterior ao envio continuará sendo o único momento de descarte pelo convidado.

**Erros previstos**

- Tentativa de acessar ou excluir um registro já enviado.
- Tentativa de iniciar novo registro fora do prazo.

**Prioridade:** alta

---

#### FR-016 Travamento dos dados do evento

Impedir alterações estruturais após o primeiro registro recebido com sucesso.

**Fluxo principal**

- Ao confirmar o primeiro registro, travar nome, capa, mensagem, data e horário do evento.
- Bloquear geração e aceitação de convites.
- Preservar apenas as alterações de acesso permitidas pelo FR-017.

**Fluxos alternativos e exceções**

- Envio com falha não contará como primeiro registro recebido.
- Convite ainda válido deixará de poder ser aceito após o travamento.
- A regra valerá igualmente em homologação e produção.

**Erros previstos**

- Alteração ou aceite de convite após o travamento.
- Operações concorrentes que tentem contornar a regra.

**Prioridade:** alta

---

#### FR-017 Acesso imediato e abertura futura

Controlar quando os responsáveis podem acessar registros e informações de recebimento.

**Fluxo principal**

- Adotar abertura futura como padrão.
- Sugerir um ano após a data do evento.
- Para evento em 29 de fevereiro, sugerir 28 de fevereiro do ano seguinte.
- Liberar à meia-noite da data escolhida, em Brasília.
- Exigir que a abertura futura seja posterior ao dia atual e ao encerramento dos envios.
- Aplicar a mesma configuração a Depoimentos e Momentos.

**Fluxos alternativos e exceções**

- Durante a janela de envios, permitir troca livre entre imediato e futuro.
- Ao encerrar a janela, travar uma data futura já configurada.
- Se o evento encerrar no modo imediato, permitir uma única mudança posterior para data futura, que ficará travada.
- A tolerância até as 13h não estenderá a janela de alteração livre.
- Ao mudar de imediato para futuro, bloquear todos os registros, inclusive os já visualizados.
- O sistema não poderá desfazer o conhecimento de conteúdo visto durante o modo imediato.

**Erros previstos**

- Data inválida, passada ou incompatível com o encerramento.
- Antecipação, adiamento ou retorno ao imediato após a trava.
- Tentativa de alteração sem vínculo com o evento.

**Prioridade:** alta

---

#### FR-018 Experiência durante o bloqueio

Preservar a surpresa até a abertura programada.

**Fluxo principal**

- Mostrar data de abertura e contagem regressiva em dias, horas e minutos.
- Ocultar conteúdo, quantidade de registros e remetentes.
- Impedir acesso por chamadas diretas à API e URLs de mídia.
- Liberar o acesso ao atingir a data configurada.

**Fluxos alternativos e exceções**

- Logs e alertas operacionais não deverão revelar conteúdo ou identidades de convidados.
- O acesso técnico habitual aos arquivos na AWS será bloqueado por permissões.
- Não será exigida proteção contra alteração deliberada dessas permissões pelo administrador.

**Erros previstos**

- Acesso antes da liberação.
- Exposição por miniaturas, listagens, cache ou URLs anteriormente emitidas.
- Exposição de informações de recebimento na interface bloqueada.

**Prioridade:** alta

---

#### FR-019 Galeria e sequência de visualização

Permitir revisitar os registros após sua liberação.

**Fluxo principal**

- Disponibilizar galeria e visualização em sequência.
- Oferecer filtros Depoimentos, Momentos e Tudo.
- Ordenar do primeiro envio ao último.
- Mostrar nome ou apelido e legenda quando houver.
- Exibir fotos de conjuntos como itens separados.
- Exigir Próximo para avançar.
- Iniciar áudio e vídeo somente após Reproduzir.

**Fluxos alternativos e exceções**

- Não avançar automaticamente ao terminar áudio ou vídeo.
- Não incluir busca por nome.
- Não incluir download ou exclusão.
- Registros continuarão disponíveis enquanto a plataforma for mantida.

**Erros previstos**

- Acesso sem autorização ou antes da liberação.
- Mídia ainda em processamento.
- Falha temporária de reprodução.

**Prioridade:** alta

---

#### FR-020 Processamento assíncrono de mídias

Preparar mídias em segundo plano sem comprometer originais recebidos.

**Fluxo principal**

- Encaminhar tarefas elegíveis ao SQS.
- Processar com novas tentativas automáticas.
- Exibir Processando somente a responsáveis com acesso liberado.
- Disponibilizar a reprodução após sucesso.

**Fluxos alternativos e exceções**

- Preservar o arquivo original após falhas de processamento.
- Após esgotar as tentativas, sinalizar indisponibilidade temporária e enviar alerta técnico.
- Permitir processamento após o prazo final quando o recebimento integral ocorreu dentro dele.
- Quantidade de tentativas e procedimentos de reprocessamento serão detalhados no HLD.

**Erros previstos**

- Falha de conversão ou preparação.
- Mensagem repetida.
- Processamento interrompido ou permanentemente malsucedido.

**Prioridade:** alta

---

#### FR-021 E-mails transacionais

Enviar convites, recuperação de senha e avisos de liberação pelo SES.

**Fluxo principal**

- Encaminhar recuperação e convite ao SES em até 1 minuto.
- Encaminhar aviso de liberação aos responsáveis em até 5 minutos após a abertura programada.
- Incluir no aviso um link para o evento, exigindo autenticação.
- Processar envios em segundo plano com novas tentativas.

**Fluxos alternativos e exceções**

- A liberação não dependerá da chegada do e-mail.
- A meta se refere ao encaminhamento ao SES; a entrega depende também do provedor do destinatário.
- Mensagens serão em português do Brasil.

**Erros previstos**

- Falha no serviço de e-mail.
- Destinatário inválido ou rejeição de entrega.
- Repetição de tarefa que possa causar mensagens duplicadas.

**Prioridade:** alta

---

#### FR-022 Monitoramento, auditoria e alertas

Permitir operação por um único responsável com informações suficientes para detectar e investigar falhas.

**Fluxo principal**

- Registrar erros, tempos de resposta e alterações sensíveis.
- Registrar autoria e momento das alterações de acesso ao evento.
- Manter registros técnicos e de auditoria por 180 dias.
- Detectar indisponibilidade e disparar alerta por e-mail em até 2 minutos.
- Alertar quando houver 3 envios com erro em uma janela de 5 minutos.
- Alertar sobre falhas definitivas de processamento.

**Fluxos alternativos e exceções**

- Não incluir senhas, textos de depoimentos ou conteúdo de mídias.
- Limiares complementares e tratamento de alertas repetidos serão definidos no HLD.
- Alertas não concederão acesso às lembranças bloqueadas.

**Erros previstos**

- Falha na coleta de métricas.
- Falha no canal de alerta.
- Exposição acidental de dados sensíveis.

**Prioridade:** alta

---

### Requisitos não funcionais

Performance

- Página dos convidados pronta para uso em até 3 segundos em conexão 4G estável.
- Perfil de rede, dispositivos e condições de medição serão especificados na validação.
- APIs de operações comuns com p95 de até 500 ms sob a carga prevista, excluindo transferência de mídias.
- Suportar 200 convidados enviando simultaneamente, distribuídos em dois eventos de 100 convidados.
- Preparar pelo menos 95% das mídias em até 5 minutos após confirmação.
- O perfil de arquivos usado no teste de carga deverá ser explicitado no HLD; não assumir apenas arquivos pequenos.

Disponibilidade

- Meta de 99,9% de disponibilidade mensal em produção.
- Detectar e disparar alertas de indisponibilidade em até 2 minutos.
- Tempo máximo de recuperação de 10 minutos nos cenários cobertos.
- Indisponibilidade de uma região inteira não terá contingência multirregião.
- RTO e disponibilidade são metas distintas e deverão ser validadas separadamente.

Segurança e autorização

- Autorizar operações no backend, verificando vínculo com cada evento.
- Proteger sessões, senhas, tokens de recuperação e convites.
- Armazenar senhas com hash apropriado, nunca em texto puro.
- Usar comunicação protegida por HTTPS.
- Manter mídias privadas e emitir autorizações temporárias restritas.
- Validar conteúdo e limites no backend, além das verificações de interface.
- Aplicar limites de requisições e tentativas de login compatíveis com a carga.
- Bloquear acesso habitual direto aos arquivos por credenciais técnicas.
- Não exigir resistência contra o administrador deliberadamente alterando permissões.
- Registrar a senha mínima de 6 caracteres como decisão com risco residual.

Observabilidade

- Registrar erros, latência e alterações sensíveis, com correlação suficiente para investigação.
- Manter registros por 180 dias.
- Alertar indisponibilidade, recorrência de falhas de envio e falhas definitivas de processamento.
- Não registrar conteúdos, credenciais ou tokens utilizáveis.
- Ferramentas e detalhamento de métricas serão definidos no HLD.

Confiabilidade e integridade de dados

- Meta de RPO zero para registros confirmados nos cenários de falha cobertos.
- Confirmação somente após persistência dos arquivos e dados necessários.
- Atomicidade lógica para conjuntos de fotos.
- Novas tentativas de envio e processamento não poderão duplicar registros.
- Original preservado quando houver falha na preparação da mídia.
- Validar recuperação dentro de uma região AWS.
- Detalhar cenários cobertos e comprovar RPO/RTO em testes; a definição da meta não constitui garantia técnica já demonstrada.
- Preservação de rascunhos limitada à permanência na página.
- Retenção das lembranças por tempo indeterminado enquanto a plataforma for mantida.

Compatibilidade e portabilidade

- Interface responsiva para celular e computador.
- Suportar as duas versões principais mais recentes de Chrome, Safari, Edge e Firefox, conforme plataformas aplicáveis.
- Testar Android, iPhone e computadores.
- Interface e e-mails exclusivamente em português do Brasil.
- API REST com JSON.
- Captura e reprodução deverão ser validadas nos navegadores suportados.
- Formatos, codecs e versões de React, .NET e PostgreSQL serão definidos no HLD e nos guidelines.

Compliance

- Revisão de privacidade obrigatória antes do lançamento público.
- Abranger aviso aos convidados, participação de menores, retenção e pedidos de exclusão fora da interface.
- Definir procedimento para esses pedidos sem acrescentar exclusão pela interface à primeira versão.
- A concordância no navegador não será tratada, isoladamente, como comprovação de atendimento a todas as exigências aplicáveis.
- Eventuais ajustes resultantes dessa revisão deverão ser incorporados antes do lançamento.

Acessibilidade no frontend consumidor

- Meta WCAG 2.2 nível AA, com verificação automatizada e manual.
- Validar navegação por teclado, contraste, identificação de campos e uso com leitores de tela.
- Evitar dependência exclusiva de cor ou ícones sem identificação.
- Apresentar mensagens compreensíveis e ações claras.
- Validar usabilidade com pessoas de diferentes idades, incluindo pessoas idosas.
- Detalhar na revisão de acessibilidade como atender aos critérios aplicáveis às mídias enviadas por convidados, sem assumir conformidade apenas pela interface.
- Definir após o protótipo a meta de conclusão sem ajuda.

---

### Arquitetura e abordagem

Abordagem

- Um único backend .NET organizado em módulos.
- Frontend React consumindo API REST/JSON.
- Separação lógica e autorização por evento.
- Upload direto ao S3 mediante autorização temporária do backend.
- Processamento elegível em segundo plano por filas SQS.
- Não adotar microsserviços nesta versão.
- HLD deverá demonstrar a viabilidade das metas, comparar alternativas de implantação e estimar custos antes da implementação da infraestrutura.

Componentes

- Frontend React para responsáveis e convidados.
- Backend .NET para contas, eventos, convites, regras temporais, autorização e confirmação dos registros.
- PostgreSQL para dados estruturados.
- S3 privado para fotos, vídeos e áudios.
- Consumidores de tarefas assíncronas da mesma solução.
- SQS para desacoplamento e novas tentativas.
- SES para e-mails transacionais.
- Monitoramento, auditoria e alertas, com serviços específicos definidos no HLD.
- Recursos separados de homologação e produção na mesma conta AWS.

Integrações

- Navegador com câmera, microfone e seleção de arquivos.
- Backend com PostgreSQL, S3, SQS e SES.
- Navegador com S3 mediante autorização temporária.
- Trello como ferramenta de organização do trabalho, utilizando o board Scrap Book já existente; nenhuma integração funcional com o produto é exigida.

### Decisões e trade-offs

#### Decisão: plataforma web com prioridade para celular
- **Justificativa:** permitir participação por link ou QR code sem instalar aplicativo.
- **Trade-off:** captura e persistência local dependem das capacidades e restrições do navegador.

#### Decisão: React, .NET, PostgreSQL e AWS
- **Justificativa:** tecnologias definidas pelo responsável pelo projeto.
- **Trade-off:** implementação e operação deverão respeitar as características desses serviços; escolhas de versões e hospedagem permanecem para o HLD.

#### Decisão: backend único organizado em módulos
- **Justificativa:** reduzir complexidade inicial para uma equipe de uma pessoa.
- **Trade-off:** módulos compartilharão parte do ciclo de implantação e dos recursos operacionais.

#### Decisão: envio direto ao S3 e processamento por SQS
- **Justificativa:** reduzir a passagem de arquivos grandes pela API e executar tarefas demoradas em segundo plano.
- **Trade-off:** exige controle de estados, autorizações temporárias, idempotência e tratamento de arquivos incompletos.

#### Decisão: uma região e uma conta AWS
- **Justificativa:** simplificar a fase inicial.
- **Trade-off:** não haverá contingência regional; isolamento entre ambientes exigirá atenção mesmo dentro da mesma conta.

#### Decisão: cadastro público e múltiplos eventos
- **Justificativa:** permitir testes externos e uso em diferentes ocasiões.
- **Trade-off:** aumenta a necessidade de isolamento, proteção contra abuso e controle de custos.

#### Decisão: participação sem login e com nome ou apelido
- **Justificativa:** reduzir atrito para convidados.
- **Trade-off:** identidade e participação única não poderão ser comprovadas; quem possuir o link poderá enviar durante a janela.

#### Decisão: senha mínima de 6 caracteres, com maiúscula e minúscula, sem segunda etapa
- **Justificativa:** preferência por simplicidade nesta versão.
- **Trade-off:** proteção mais limitada contra comprometimento de contas, mesmo com limites de tentativas.

#### Decisão: não confirmar e-mail no cadastro
- **Justificativa:** simplificar o fluxo inicial.
- **Trade-off:** endereços digitados incorretamente ou não controlados pelo usuário podem comprometer comunicações e recuperação.

#### Decisão: mudanças de abertura permitidas durante a janela
- **Justificativa:** permitir conferência dos registros no modo imediato antes de preservar a surpresa.
- **Trade-off:** conteúdo já visto não volta a ser desconhecido; a configuração fica sujeita às regras de travamento após o encerramento.

#### Decisão: retenção sem prazo final definido
- **Justificativa:** permitir revisitar as lembranças enquanto a plataforma existir.
- **Trade-off:** custos e responsabilidade de manutenção se acumulam, com política de privacidade ainda sujeita a revisão.

#### Decisão: sem edição ou exclusão de lembranças pela interface
- **Justificativa:** manter o escopo inicial simples e preservar os registros enviados.
- **Trade-off:** correções e solicitações de privacidade exigirão tratamento fora da interface.

---

### Dependências

#### Técnica: arquitetura e dimensionamento

Diórgenes Backes deverá concluir o HLD, incluindo hospedagem, redundância, recuperação, formatos de mídia, custos e evidências necessárias para validar as metas.

#### Técnica: infraestrutura AWS

Será necessário configurar região, ambientes isolados, permissões, banco, armazenamento, filas, envio de e-mails, monitoramento e recuperação.

#### Técnica: domínio e envio de e-mails

Será necessário definir domínio, HTTPS e configuração do SES compatível com os destinatários e o volume esperado.

#### Organizacional: protótipo e participantes

Diórgenes já possui pessoas disponíveis para testes. O protótipo deverá ser avaliado com diferentes idades e níveis de familiaridade tecnológica.

#### Externa: conexão do local

O local terá Wi-Fi. Sua capacidade deverá ser validada antes do evento, considerando uso compartilhado e uploads simultâneos.

#### Organizacional: revisão de privacidade e acessibilidade

Antes do lançamento, será necessário definir os procedimentos de privacidade e resolver os requisitos de acessibilidade aplicáveis, incluindo conteúdo de mídia.

#### Organizacional: disponibilidade do responsável

Diórgenes Backes será o único responsável pelo desenvolvimento e pela operação. O planejamento deverá considerar essa capacidade.

#### Organizacional: orçamento

Não existe orçamento mensal definido. A estimativa será produzida no HLD, considerando produção, homologação, retenção e tráfego.

---

### Riscos e mitigação

#### Conexão instável impedir ou atrasar envios
- **Probabilidade:** média
- **Impacto:** alto; perda da oportunidade de registrar ou enviar lembranças.
- **Mitigação:**
  - Validar o Wi-Fi antes do evento.
  - Preservar conteúdo na página para nova tentativa.
  - Orientar uso de internet móvel quando o problema for a conexão local.
- **Plano de contingência:** gravar pelo celular e enviar quando a conectividade ou a plataforma voltar, respeitando o prazo original.

#### Acesso indevido ou antecipado aos registros
- **Probabilidade:** média
- **Impacto:** alto; exposição de dados e quebra da surpresa.
- **Mitigação:**
  - Validar autorização por evento e data no backend.
  - Manter mídias privadas.
  - Testar URLs, caches, listagens e permissões técnicas.
  - Auditar mudanças sensíveis.
- **Plano de contingência:** bloquear o caminho afetado, revogar acessos utilizáveis e corrigir a falha antes de restabelecer o acesso; exposição já ocorrida não poderá ser desfeita.

#### Perda de registros confirmados
- **Probabilidade:** baixa, como hipótese inicial
- **Impacto:** crítico; perda de lembranças insubstituíveis.
- **Mitigação:**
  - Validar persistência antes da confirmação.
  - Projetar redundância e backups.
  - Testar recuperação e integridade dos dados.
  - Preservar originais durante processamento.
- **Plano de contingência:** executar o procedimento de recuperação e verificar registros restaurados; se os testes não demonstrarem RPO zero nos cenários cobertos, bloquear o lançamento até corrigir ou revisar formalmente a meta.

#### Atrasos e dificuldade de operação por haver um único responsável
- **Probabilidade:** média
- **Impacto:** alto; comprometimento do prazo ou resposta lenta a incidentes.
- **Mitigação:**
  - Desenvolver incrementalmente.
  - Automatizar verificações e implantação.
  - Preparar procedimentos de recuperação.
  - Usar alertas objetivos.
- **Plano de contingência:** repriorizar o planejamento e negociar explicitamente mudanças de escopo ou prazo, sem retirar silenciosamente requisitos obrigatórios.

#### Custos e metas técnicas incompatíveis com a fase inicial
- **Probabilidade:** média, hipótese
- **Impacto:** alto; infraestrutura acima do orçamento ou metas não atendidas.
- **Mitigação:**
  - Estimar custos no HLD.
  - Executar testes de carga e recuperação antes do lançamento.
  - Acompanhar armazenamento e tráfego.
- **Plano de contingência:** revisar arquitetura, orçamento ou metas com aprovação de Diórgenes antes da produção.

#### Falhas de processamento ou incompatibilidade de mídia
- **Probabilidade:** média, hipótese
- **Impacto:** alto; arquivos salvos, mas temporariamente indisponíveis para reprodução.
- **Mitigação:**
  - Definir formatos suportados.
  - Testar dispositivos e navegadores.
  - Usar novas tentativas e preservar originais.
- **Plano de contingência:** sinalizar indisponibilidade, alertar o responsável técnico e reprocessar após correção.

#### Uso abusivo do cadastro ou dos envios públicos
- **Probabilidade:** média, hipótese
- **Impacto:** alto; custos, indisponibilidade ou registros indesejados.
- **Mitigação:**
  - Aplicar limites de requisições e login.
  - Validar arquivos, prazos e autorizações temporárias.
  - Monitorar padrões anormais.
- **Plano de contingência:** aplicar bloqueios técnicos direcionados e ajustar controles sem comprometer desnecessariamente os eventos legítimos.

#### Requisitos de privacidade ou acessibilidade permanecerem sem solução
- **Probabilidade:** média, hipótese
- **Impacto:** alto; impedir lançamento público conforme os critérios definidos.
- **Mitigação:**
  - Concluir revisões antes do lançamento.
  - Definir tratamento de menores e solicitações de exclusão.
  - Verificar critérios de acessibilidade aplicáveis às mídias.
- **Plano de contingência:** corrigir os pontos identificados e manter o lançamento bloqueado enquanto houver pendências impeditivas.

---

### Critérios de aceitação

Checklist objetivo que define se a feature está pronta.

- [ ] Cadastro público, login, edição e recuperação de conta funcionam conforme as regras.
- [ ] Links de recuperação expiram em 30 minutos e são de uso único.
- [ ] Troca ou recuperação de senha encerra todas as sessões.
- [ ] Contas podem criar e participar de vários eventos sem acesso aos eventos de terceiros.
- [ ] Convites expiram em 24 horas, são de uso único e são invalidados por substituição.
- [ ] O primeiro registro confirmado trava os dados do evento e os convites.
- [ ] Evento funciona com um ou dois responsáveis, com permissões iguais.
- [ ] Página pública apresenta personalização, opções de envio e QR code.
- [ ] QR code para impressão aponta para o evento correto.
- [ ] Janela abre duas horas antes e encerra novos envios ao meio-dia seguinte, em Brasília.
- [ ] Somente envios iniciados no prazo podem continuar até as 13h.
- [ ] Arquivos integralmente recebidos no prazo podem ser processados depois.
- [ ] Nome e concordância são reutilizados por evento no navegador.
- [ ] Alterar o nome exige nova concordância.
- [ ] Todos os formatos e limites aprovados são validados no frontend e backend.
- [ ] Gravações encerram no limite e seguem para revisão, sem envio automático.
- [ ] Campos de texto e legenda respeitam limites e avisos progressivos.
- [ ] Conjuntos de fotos têm sucesso ou falha integral.
- [ ] Novas tentativas não duplicam registros.
- [ ] Confirmação só aparece após persistência completa.
- [ ] Convidados não visualizam nem excluem registros enviados.
- [ ] Regras de mudança entre imediato e futuro são aplicadas também no backend.
- [ ] Conteúdo, quantidade e remetentes não são expostos antes da liberação.
- [ ] Contagem regressiva mostra dias, horas e minutos.
- [ ] Galeria e sequência respeitam filtros e ordem cronológica.
- [ ] Fotos de conjuntos aparecem separadamente com a mesma legenda.
- [ ] Reprodução e avanço dependem de ações explícitas.
- [ ] Não existem ações de download ou exclusão de lembranças na interface.
- [ ] Mídias em processamento são identificadas somente quando o acesso estiver liberado.
- [ ] Falhas definitivas preservam originais e geram alerta.
- [ ] E-mails são encaminhados ao SES dentro dos prazos definidos.
- [ ] A solução suporta 200 envios simultâneos no perfil de teste documentado.
- [ ] APIs comuns atendem p95 de até 500 ms sob a carga prevista.
- [ ] A página atende à meta de carregamento no perfil 4G documentado.
- [ ] Pelo menos 95% das mídias ficam prontas em até 5 minutos.
- [ ] Testes de recuperação demonstram até 10 minutos de RTO e RPO zero nos cenários cobertos.
- [ ] Monitoramento de disponibilidade mensal está configurado para avaliar a meta de 99,9%.
- [ ] Alertas e retenção de registros técnicos por 180 dias estão configurados e testados.
- [ ] Os navegadores e dispositivos previstos passam pelos fluxos críticos.
- [ ] A revisão de acessibilidade verifica os critérios aplicáveis de WCAG 2.2 AA.
- [ ] A revisão de privacidade e seus procedimentos estão concluídos.
- [ ] Protótipo foi testado e a meta de conclusão sem ajuda foi definida e incorporada.
- [ ] Homologação e produção mantêm recursos e dados isolados.
- [ ] Todos os critérios aplicáveis à entrega estão aprovados.
- [ ] Não há defeitos críticos ou de alta gravidade em aberto.
- [ ] Testes controlados em produção foram concluídos antes do prazo de lançamento.

---

### Testes e validação

Tipos de teste obrigatórios

- Testes unitários das regras de prazo, limites, convites, sessões e bloqueios.
- Testes de integração de banco, S3, SQS, SES e autenticação.
- Testes completos dos fluxos de responsáveis e convidados.
- Testes de segurança de autorização, isolamento, tokens, arquivos e acesso temporal.
- Testes de carga com 200 envios simultâneos.
- Testes de integridade e atomicidade de conjuntos de fotos.
- Testes de recuperação de falhas, redundância e restauração.
- Testes de processamento assíncrono, duplicações e novas tentativas.
- Testes de acessibilidade automatizados e manuais.
- Testes de usabilidade com participantes reais de diferentes idades.
- Testes de compatibilidade nos navegadores e dispositivos previstos.
- Testes controlados de funcionamento em produção.

Estratégia de validação

- Validar localmente antes de cada promoção para homologação.
- Executar os fluxos completos em homologação com eventos independentes e as mesmas regras de produção.
- Testar fronteiras de horário: abertura, meio-dia, limite das 13h e meia-noite da liberação.
- Testar recebimento concorrente do primeiro registro com edição ou aceite de convite.
- Testar troca de imediato para futuro, incluindo URLs e sessões já existentes.
- Simular falhas de conexão, resposta perdida, upload parcial e processamento interrompido.
- Medir desempenho com mistura documentada de tamanhos e formatos.
- Executar testes de recuperação dentro dos cenários regionais definidos no HLD.
- Usar testes do protótipo para definir a meta de envio sem ajuda.
- Avaliar o Wi-Fi no local e ensaiar as orientações de contingência.
- Promover para produção somente com critérios aprovados e sem defeitos críticos ou de alta gravidade.
- Executar testes controlados em produção usando eventos próprios de teste, sem interferir nas lembranças reais.
- Após o casamento piloto, avaliar participação aproximada e coletar feedback para decidir a evolução do produto.
