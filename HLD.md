### HLD: Scrap Book

Versão: 1.1
Data: 2026-11-13
Responsável: Diórgenes Backes

Adendo de organização — 2026-11-11: por definição posterior do responsável, a estrutura passa a seis componentes com a inclusão de `scrapbook-infrastructure`. A criação dessa área não altera a configuração manual da infraestrutura nem as demais decisões de implantação deste HLD.

Adendo de dados — 2026-11-11: por definição posterior do responsável, todas as tabelas terão PK inteira simples `id`, com padrão `bigint GENERATED ALWAYS AS IDENTITY`. Essa decisão substitui UUID como PK relacional. Identificadores técnicos de mensagem, correlação e idempotência mantêm os formatos definidos em seus contratos. Nomenclatura, repetição segura e rollback obrigatório seguem o [guideline SQL](guidelines/sql.md).

Adendo de arquitetura — 2026-11-13: o responsável determinou a retirada da modularização do backend. O serviço passa a Clean Architecture por camadas/conceitos, com Controllers MVC, sufixos obrigatórios, repositórios e ScrapBookDbContext único. Essa decisão substitui a organização modular da versão 1.0; veja [ADR-001](adr/ADR-001-backend-clean-architecture.md). Os nomes das capacidades de negócio e os schemas existentes não representam módulos de código. As demais decisões de infraestrutura e contratos permanecem sujeitas às validações registradas.

---

### Objetivo técnico

Estruturar o Scrap Book como uma aplicação web com backend em Clean Architecture, capaz de receber registros de eventos, preservar arquivos originais, preparar mídias para reprodução e controlar o acesso conforme permissões e datas de abertura.

A solução será desenvolvida do zero. A arquitetura deverá suportar 200 envios simultâneos, separar dados de eventos e ambientes, executar tarefas assíncronas com recuperação de falhas e permitir operação por um único responsável.

As metas do PRD serão referências de validação, especialmente p95 de até 500 ms nas operações comuns, disponibilidade mensal de 99,9%, recuperação em até 10 minutos e nenhuma perda de registros confirmados nos cenários de falha cobertos. A arquitetura não será considerada validada apenas pela escolha dos serviços: essas metas dependerão de testes.

Dependências com outros sistemas

- GitHub e GitHub Actions para código, validações e publicação.
- Amazon Cognito para identidade e autenticação.
- Amazon SES para e-mails transacionais.
- Amazon SNS, SQS e EventBridge para distribuição, agendamento e processamento de mensagens.
- AWS Elemental MediaConvert para preparação de vídeos e áudios.
- Amazon CloudWatch e AWS X-Ray para observabilidade.
- Trello como ferramenta de planejamento, sem integração funcional obrigatória com o produto.

---

### Arquitetura geral

O backend será um serviço .NET simples em Clean Architecture, organizado por camadas/conceitos. Suas capacidades de negócio incluem:

- Contas e Acesso.
- Eventos e Convites.
- Registros e Mídias.
- Notificações, como capacidade de apoio.

API, produtores e consumidores de mensagens executarão na mesma aplicação e no mesmo serviço ECS. Não haverá um serviço genérico de workers separado dos domínios. As réplicas compartilharão o banco e disputarão mensagens das respectivas filas.

Domain, Application, Infrastructure e API contêm diretórios por conceito, sem Modules ou subpastas funcionais. Controllers chamam serviços de Application; contratos de repositório são implementados em Infrastructure. Processos com consistência eventual continuam usando mensagens.

Operações locais usam o ScrapBookDbContext scoped e transações curtas, incluindo Outbox quando aplicável. Integrações externas não participam dessas transações; não se exige transação distribuída.

Ambiente de implantação

- Cloud AWS, exclusivamente em Norte da Virgínia, `us-east-1`.
- Homologação e produção na mesma conta, em VPCs separadas.
- Frontend estático no Amplify Hosting.
- ALB público com HTTPS e AWS WAF.
- Fargate em sub-redes públicas, com IP público e entrada permitida somente pelo security group do ALB na porta da aplicação.
- RDS em sub-redes privadas, sem acesso público.
- Gateway Endpoint para S3 em cada VPC.
- Sem NAT Gateway na configuração inicial.
- Produção com no mínimo duas tarefas Fargate em duas zonas e escalabilidade automática.
- Homologação com uma tarefa normalmente, ampliada para os testes necessários.
- RDS PostgreSQL Multi-AZ com uma standby em produção.
- RDS Single-AZ em homologação, convertido para Multi-AZ durante testes de recuperação.
- Infraestrutura criada e configurada manualmente por Diórgenes, com documentação reproduzível.
- Sem recuperação multirregião nesta versão.

Tecnologias principais

- React e TypeScript.
- .NET e Entity Framework Core.
- PostgreSQL gerenciado pelo Amazon RDS.
- Liquibase para alterações estruturais do banco.
- ECS Fargate e ECR.
- Amplify Hosting, S3 e CloudFront.
- Cognito, Secrets Manager e KMS.
- SQS Standard, SNS, EventBridge e EventBridge Scheduler.
- SES e MediaConvert.
- CloudWatch, OpenTelemetry e X-Ray.
- Docker Compose para desenvolvimento local.
- Playwright para fluxos completos e k6 para carga e performance.

Padrões adotados

- Clean Architecture em serviço único, com diretórios/sufixos por conceito e Controllers MVC.
- REST/JSON com prefixo `/api/v1` e documentação OpenAPI.
- Backend intermediando autenticação e sessões do frontend.
- Outbox transacional local.
- Consumidores idempotentes.
- Consistência eventual entre processos/capacidades quando aprovada.
- Contratos de mensagens versionados.
- Upload direto ao S3, com autorização temporária.
- Paginação por cursor.
- Processamento assíncrono e DLQs.
- Encerramento controlado das tarefas.

---

### Componentes e responsabilidades

| Componente | Responsabilidades | Dependências |
| ----------- | ----------------- | ------------ |
| Frontend React | Navegação, captura, revisão, upload, acompanhamento de status e visualização | API, Cognito gerenciado, S3 e CloudFront |
| Amplify Hosting | Hospedar e distribuir os arquivos estáticos do frontend | Artefatos publicados pelo GitHub Actions |
| ALB e WAF | Terminar HTTPS, encaminhar tráfego e aplicar proteção de entrada | Serviço ECS, certificados e regras de rede |
| Contas e Acesso | Vincular identidade Cognito, administrar sessões e integrar operações de conta | Cognito, PostgreSQL, Secrets Manager e proteção criptográfica |
| Eventos e Convites | Configurar eventos, responsáveis, convites, janela de envio e abertura | PostgreSQL, contratos internos e mensageria |
| Registros e Mídias | Controlar uploads, validações, registros, arquivos, variantes e reprodução | PostgreSQL, S3, CloudFront, SQS e MediaConvert |
| Notificações | Preparar e enviar e-mails, controlar tentativas e registrar resultados | PostgreSQL, SQS e SES |
| Publicador de Outbox | Publicar mensagens persistidas e retomar falhas de publicação | PostgreSQL, SNS e SQS |
| Consumidores internos | Executar tarefas dos módulos com idempotência e concorrência limitada | Filas, banco e integrações do módulo |
| RDS PostgreSQL | Persistir dados estruturados, sessões, estados e Outbox | Rede privada e backups |
| S3 de capas | Armazenar capas distribuídas publicamente pelo CloudFront | KMS e permissões de acesso |
| S3 de lembranças | Armazenar originais, temporários e variantes privadas | KMS, versionamento e políticas de limpeza |
| CloudFront | Distribuir capas e mídias; proteger lembranças com URLs assinadas | S3 privado e mecanismo de assinatura |
| MediaConvert | Gerar versões de reprodução de vídeos e áudios | S3, IAM e EventBridge |
| SNS | Distribuir eventos internos e alertas operacionais | Assinaturas SQS e assinatura de e-mail operacional |
| SQS e DLQs | Armazenar tarefas, permitir novas tentativas e preservar falhas | Produtores e consumidores |
| EventBridge | Encaminhar resultados do MediaConvert | MediaConvert e SQS |
| EventBridge Scheduler | Agendar processamento diário dos avisos de abertura | SQS |
| Liquibase em tarefa Fargate | Aplicar alterações e rollbacks de banco acionados pelo pipeline | GitHub Actions, ECR, Secrets Manager e RDS |
| Observabilidade | Centralizar logs, métricas, rastros, painéis e alarmes | CloudWatch, OpenTelemetry, X-Ray e SNS |

---

### Fluxo de requisições e de dados

**Fluxo de requisição**

- O navegador carrega a aplicação estática pelo Amplify Hosting.
- As chamadas de negócio chegam à API pelo ALB, passando pelo WAF.
- A API valida entrada, sessão, origem e autorização aplicável.
- O caso de uso é executado no módulo responsável.
- O módulo acessa seus dados pelo Entity Framework Core.
- Operações entre módulos usam contratos internos ou mensagens, conforme a consistência necessária.
- A resposta contém dados autorizados ou erro padronizado, sem detalhes internos.

Autenticação

- O backend inicia o login nas telas gerenciadas pelo Cognito.
- O usuário autentica no User Pool do ambiente.
- O backend conclui o fluxo, valida a resposta e cria a sessão local.
- Tokens necessários ficam criptografados no PostgreSQL.
- O navegador recebe somente um identificador aleatório em cookie seguro e HttpOnly.
- Todas as tarefas Fargate consultam a mesma fonte de sessões.
- Cadastro e alteração de e-mail exigem confirmação do endereço.
- Recuperação segue o Cognito: código por e-mail válido por uma hora.
- A implementação deverá coordenar sessões locais e identidade Cognito para cumprir a revogação após troca ou recuperação de senha.

Envio de registro

- O convidado identifica-se, fornece concordância, captura ou seleciona conteúdo e revisa.
- Ao tocar em Enviar, o frontend solicita uma Sessão de Upload com chave de idempotência.
- O backend deve receber essa solicitação antes do encerramento; horário do navegador não comprova elegibilidade.
- A aplicação valida evento, janela, quantidade e metadados declarados.
- O backend registra arquivos esperados e prazo, emitindo autorizações temporárias de envio.
- O navegador envia os arquivos diretamente para a área temporária privada do S3.
- Vídeos grandes utilizam multipart e repetem somente partes que falharem.
- O backend valida o conteúdo efetivo, incluindo tamanho, formato e duração.
- O frontend consulta o status a cada 5 segundos enquanto houver etapa pendente.
- O Registro é confirmado somente após persistência completa dos arquivos e dados necessários.
- A transação local persiste também as mensagens de Outbox.
- O convidado recebe a confirmação e não obtém acesso posterior às mídias.
- Para conjuntos de fotos, a confirmação é integral: falhar uma foto impede confirmar o conjunto.

Prazos

- Novos envios abrem duas horas antes do evento e encerram ao meio-dia seguinte.
- Envios autorizados antes do encerramento podem concluir até as 13h.
- Recebimento integral dentro do prazo permite processamento posterior.
- A implementação deverá registrar evidências de recebimento no servidor; processamento atrasado não deverá ser confundido com upload atrasado.
- Arquivos incompletos ou rejeitados entram em limpeza após 24 horas. O mecanismo e o atraso operacional dessa limpeza serão validados.

Primeiro registro e travamento

- Registros e Mídias confirma o envio e persiste o evento interno na Outbox.
- O publicador encaminha a mensagem.
- Eventos e Convites consome a ocorrência e trava os dados e convites.
- A meta aprovada é concluir o travamento em até 5 segundos em condições normais.
- Existe uma janela aceita em que alterações ou convites ainda podem ocorrer.
- Falhas provocam novas tentativas; mensagens esgotadas seguem para DLQ.

Visualização

- A API consulta sessão, vínculo com o evento e configuração de abertura em cada pedido.
- Não depende do agendador para liberar conteúdo.
- Eventos bloqueados retornam somente informações permitidas, como data e contagem regressiva.
- A galeria retorna até 10 itens com cursor estável.
- Cada foto de um conjunto conta como um item independente.
- A API emite URLs assinadas de até 2 minutos para mídias autorizadas.
- Renovação exige nova autorização.
- Vídeos e áudios começam somente por ação do usuário.
- A navegação avança somente por Próximo.
- Mudar de imediato para futuro impede novas autorizações; URLs existentes têm a tolerância aprovada. Conteúdo já entregue ao dispositivo não pode ser recolhido.

**Fluxo de dados**

- Conteúdo do convidado → S3 temporário privado → validação → original confirmado → variantes de visualização.
- Registro e Outbox → PostgreSQL → publicador → SNS/SQS → consumidor.
- Evento de domínio → SNS → fila própria de cada módulo interessado → processamento idempotente.
- Vídeo ou áudio confirmado → fila de processamento → MediaConvert → S3 → EventBridge → SQS → atualização do estado.
- Foto confirmada → consumidor .NET → JPEG e miniatura → S3 → atualização do estado.
- Agendamento diário às 00h em `America/Sao_Paulo` → SQS → busca de eventos liberados sem aviso concluído → fila de e-mails → SES.
- Logs, métricas e rastros → CloudWatch/X-Ray → alarmes → SNS → e-mail operacional.

Preparação das mídias

- Vídeos: uma versão MP4, H.264, áudio AAC, até 1080p e até 30 fps.
- Não ampliar resoluções nem taxas de quadros menores.
- Áudios: uma versão M4A com AAC.
- Fotos: JPEG até 2.048 pixels no maior lado e miniatura até 400 pixels.
- Preservar orientação, proporção e imagens menores.
- Remover localização das versões exibidas.
- Preservar os originais privados.
- Formatos iniciais: JPEG, PNG, HEIC/HEIF; MP4, MOV, WebM; MP3, M4A, WAV, OGG e WebM.
- Extensões não garantem suporte: containers, codecs e ferramentas de validação deverão ser testados.

---

### Modelo de dados (alto nível)

Detalhamento documental de 2026-11-11: [modelagem completa proposta](db/README.md), com [diagrama ER](db/DB-Diagrama.md) e [dicionário físico](db/DB-Modelo-Fisico.md). As decisões adicionais estão identificadas para revisão; não representam scripts ou persistência implementada. Este detalhamento preserva as decisões aprovadas abaixo.

Adendo de implementação — 2026-11-12: a [base Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) contém os scripts iniciais e seus rollbacks para os quatro schemas e 33 tabelas. O contrato DDL exige PostgreSQL 18 e usa 18.6 na validação local. EF, provisionamento RDS, grants DDL/DML e pipeline AWS continuam pendentes; os scripts não alteram a consistência eventual nem a propriedade dos módulos.

Entidades principais

- **Conta:** ID inteiro local, referência ao `sub` do Cognito e dados de perfil necessários.
- **Sessão de Acesso:** usuário, identificador protegido, expiração, revogação e tokens criptografados.
- **Evento:** nome, data, horário, personalização, modo de abertura, data de liberação e estado de travamento.
- **Responsável pelo Evento:** vínculo entre conta e evento, com até dois responsáveis e permissões iguais.
- **Convite:** evento, identificador protegido, validade, utilização e invalidação.
- **Registro:** envio lógico, categoria, nome ou apelido, legenda, texto quando aplicável, recebimento e estado.
- **Arquivo:** mídia pertencente ao Registro, tamanho, duração, tipo e referência ao original.
- **Variante de Arquivo:** reprodução, visualização ou miniatura, com formato e localização.
- **Sessão de Upload:** início autorizado, prazo, arquivos esperados e estado.
- **Operação Idempotente:** chave da tentativa lógica e resultado reconhecível em novas tentativas.
- **Trabalho de Processamento:** arquivo, operação, referência externa, tentativas e resultado.
- **Outbox:** mensagem pendente de publicação.
- **Mensagem Processada:** identificação usada para deduplicar efeitos do consumidor.
- **Notificação:** tipo, destinatário, evento relacionado, tentativas e resultado.
- **Auditoria:** alteração sensível, autoria, momento e correlação.

Relações

- Uma Conta pode ser responsável por vários Eventos.
- Um Evento possui um ou dois Responsáveis e vários Registros.
- Um Evento pode possuir Convites, respeitando invalidação, expiração e travamento.
- Um Registro contém texto ou um ou mais Arquivos, conforme o formato permitido.
- Um Arquivo possui seu original e pode ter várias Variantes.
- Uma Sessão de Upload representa uma tentativa lógica de recebimento.
- Um Arquivo pode ter trabalhos de processamento e novas tentativas.
- Notificações podem estar associadas a eventos, convites ou outras operações.
- Mensagens de Outbox e controles de consumo pertencem ao módulo responsável.
- Toda tabela terá PK relacional inteira `id`, com padrão `bigint GENERATED ALWAYS AS IDENTITY`; FKs terão tipos compatíveis.
- Identificadores técnicos e externos são campos distintos da PK, conforme a semântica e os contratos; UUID pode ser usado para correlação, mensagens e chaves idempotentes.
- Datas e horários de negócio serão `timestamp without time zone`, interpretados em `America/Sao_Paulo`.
- Identificadores e horários recebidos de serviços externos deverão ser normalizados explicitamente; não reinterpretar timestamps UTC como horários locais.

Estados do Registro

- Recebendo.
- Validando.
- Confirmado.
- Processando.
- Disponível.
- Rejeitado.
- Falha de processamento.

Esses estados não dispensam o registro persistente de confirmação: um arquivo em processamento já pode ter sido recebido com sucesso. Falha de processamento não transforma um envio confirmado em upload rejeitado.

Fonte de verdade

- Cognito para identidade, credenciais e verificação de e-mail.
- PostgreSQL para vínculos, permissões de negócio, sessões locais, eventos, registros e estados.
- S3 para o conteúdo dos arquivos.
- SQS e SNS são transporte, não fonte exclusiva da verdade de registros confirmados.
- Não haverá Redis ou cache distribuído de dados de negócio.
- Cada ambiente terá seu banco e um ScrapBookDbContext scoped. Schemas físicos existentes são preservados; configurações EF ficam em classes por entidade.
- Mudanças estruturais ocorrerão exclusivamente pelo pipeline Liquibase.

---

### Interfaces públicas

Os nomes abaixo representam contratos nominais. Rotas detalhadas, payloads e códigos serão especificados no OpenAPI e no LLD.

| Nome | Tipo | Protocolo | Exposição | SLAs/Limites |
| ---- | ---- | ---------- | --------- | ------------- |
| Autenticação e sessão | API | HTTPS/OIDC e REST/JSON | Externa | Sessão local até 7 dias; regras Cognito aplicáveis |
| Contas | API | REST/JSON | Externa autenticada | p95 até 500 ms para operações comuns, sujeito à validação das dependências |
| Eventos e convites | API | REST/JSON | Externa autenticada | Até dois responsáveis; convite de 24 horas e uso único |
| Página pública do evento | API | REST/JSON | Externa sem login | Sem listagem de lembranças |
| Início e conclusão de upload | API | REST/JSON | Externa com autorização de envio | Janela e tolerância verificadas pelo servidor |
| Transferência de arquivos | API | HTTPS/S3 | Externa com autorização temporária | Fotos 20 MB; vídeos 500 MB; áudios 50 MB |
| Status de envio | API | REST/JSON | Externa restrita ao envio autorizado | Consulta a cada 5 segundos; sem acesso ao conteúdo |
| Galeria | API | REST/JSON | Externa autenticada | 10 itens, cursor, filtros e autorização temporal |
| Reprodução | API | HTTPS/CloudFront | Externa com URL assinada | Validade até 2 minutos |
| Eventos internos | Tópico e filas | SNS/SQS Standard, JSON | Interna | Entrega potencialmente repetida e fora de ordem |
| Processamento de mídias | Fila | SQS Standard, JSON | Interna | Meta de 95% das mídias prontas em até 5 minutos |
| Resultados MediaConvert | Evento e fila | EventBridge/SQS | Interna | Processamento idempotente |
| E-mails | Fila | SQS Standard, JSON | Interna | Convite e recuperação: encaminhamento em até 1 minuto; abertura: até 5 minutos |
| Aviso diário de abertura | Agendamento e fila | EventBridge Scheduler/SQS | Interna | Diariamente às 00h em Brasília |
| Saúde e prontidão | API | HTTP/HTTPS | Operacional restrita | Prontidão usada pelo ALB |

As mensagens terão identificador, tipo, versão, data de ocorrência, identificador do evento quando aplicável e correlação. Transportarão referências, sem o conteúdo das mídias.

O contrato admitirá campos opcionais novos. Alterações incompatíveis exigirão nova versão e tratamento de mensagens já enfileiradas.

Erros HTTP terão código identificável, mensagem em português, detalhes por campo quando aplicável e correlação, sem stack traces ou segredos.

---

### Considerações de escalabilidade e disponibilidade

Abordagem geral

- Escalar horizontalmente o serviço ECS a partir de duas tarefas em produção.
- API e consumidores escalam juntos.
- Limitar concorrência de fotos, validações e consumo para preservar a API.
- Usar MediaConvert para retirar a conversão pesada de vídeos e áudios das tarefas.
- Definir quantidade máxima de tarefas e dimensionamento após carga.
- Não inferir capacidade do PostgreSQL apenas pela quantidade de uploads, pois os arquivos vão diretamente ao S3.
- Usar RDS Multi-AZ para recuperação de falhas de infraestrutura em produção.
- Manter backups automáticos por 35 dias e recuperação para ponto no tempo.
- Habilitar versionamento S3 para auxiliar recuperação de sobrescritas ou exclusões acidentais.
- Não considerar replicação ou versionamento suficientes para todos os cenários de erro lógico.

Técnicas aplicadas

- Balanceamento pelo ALB.
- Escalabilidade automática com métricas a definir após os testes.
- Paginação por cursor e consultas indexadas conforme os fluxos.
- Gateway Endpoint para acesso ao S3.
- Cache de arquivos estáticos no Amplify/CloudFront, sem cache distribuído de dados da aplicação.
- Outbox com recuperação após falha de publicação.
- Deduplicação e idempotência dos consumidores.
- DLQ por fila.
- Limites de concorrência e controle do volume de trabalhos em andamento.
- Renovação apropriada da visibilidade de mensagens longas.
- Encerramento controlado, interrompendo novas buscas e preservando trabalhos retomáveis.
- WAF e proteção de login, sem controles adicionais por conta ou sessão de upload nesta versão.
- Calibração de limites por IP para o Wi-Fi compartilhado do evento.
- Sem particionamento ou sharding nesta versão.

Meta de disponibilidade

- 99,9% mensal em produção.
- Detectar e disparar alerta de indisponibilidade em até 2 minutos.
- Recuperação em até 10 minutos nos cenários definidos e testados.
- RPO zero para registros confirmados nos cenários cobertos.
- p95 até 500 ms nas operações comuns.
- Página pronta em até 3 segundos no perfil 4G documentado.
- 200 envios simultâneos.
- 95% das mídias prontas em até 5 minutos.

Ressalvas de validação

- Rollback manual pode dificultar a meta de recuperação.
- Alterações incompatíveis de banco e backend poderão causar indisponibilidade entre implantações.
- Não haverá coordenação automatizada dessas releases nesta versão.
- Indisponibilidade regional está fora da recuperação automática.
- Falhas lógicas, exclusões e restaurações precisam de cenários próprios; não serão cobertas por uma promessa genérica de perda zero.
- A abertura de acesso depende da data, e não do agendador de e-mail.
- Os testes dessas condições serão obrigatórios antes do casamento, com acionamento manual.

---

### Segurança

Autenticação

- User Pool separado por ambiente.
- Telas gerenciadas do Cognito.
- Backend intermediando autenticação e mantendo tokens fora do JavaScript.
- Sessões persistidas no PostgreSQL, compartilhadas pelas tarefas.
- Cookie seguro e HttpOnly com identificador aleatório.
- Confirmação de e-mail no cadastro.
- Confirmação do novo endereço antes de substituir o anterior.
- Recuperação por código Cognito válido por uma hora.
- Política de senha do PRD, sujeita à comprovação de configuração no Cognito.
- Sem MFA nesta versão.
- Validação de origem e proteção contra CSRF.
- Fluxo OIDC, parâmetros de cookie e mecanismo de revogação serão detalhados e testados antes da implementação definitiva.

Autorização

- Verificação de vínculo e permissão no backend.
- Mesmas permissões para os dois responsáveis.
- Autorização temporal em cada pedido de acesso.
- Bloqueio de conteúdo, quantidades e remetentes antes da abertura.
- Autorizações de upload restritas aos arquivos e à operação previstos.
- URLs assinadas de reprodução com validade até 2 minutos.
- Após bloqueio, nenhuma nova URL será emitida.
- Transferências já iniciadas e dados já entregues ao navegador exigem tratamento explícito; a expiração não recolhe conteúdo recebido.
- Credenciais técnicas habituais não terão leitura direta das lembranças.
- Alteração deliberada de permissões pelo administrador permanece fora da proteção exigida.

Proteção de dados

- HTTPS nas interfaces públicas.
- Criptografia S3 com chave KMS separada por ambiente.
- Tokens Cognito criptografados no PostgreSQL.
- Buckets privados para capas e lembranças.
- Capas distribuídas publicamente por CloudFront.
- Originais e variantes das lembranças sujeitos à autorização.
- Remoção de localização das versões de fotos exibidas.
- Retenção das lembranças enquanto a plataforma for mantida.
- Limpeza de incompletos e rejeitados após 24 horas, com implementação validada.
- Registros técnicos e auditoria mantidos por 180 dias.
- Não armazenar conteúdo de depoimentos, tokens, senhas ou URLs utilizáveis em logs.
- Revisão de privacidade para menores, retenção e pedidos de exclusão fora da interface.
- Criptografia do RDS, certificados internos e parâmetros de transporte serão especificados na configuração de infraestrutura.

Gestão de segredos

- AWS Secrets Manager.
- Acesso via funções IAM das tarefas, sem credenciais permanentes no código.
- GitHub Actions autenticado na AWS por OIDC.
- Permissões específicas por repositório e ambiente.
- Chaves de proteção compartilhadas entre tarefas e protegidas pelo KMS.
- Armazenamento, rotação e retenção dessas chaves deverão preservar a leitura de sessões válidas após reinício ou implantação.
- Tarefa Liquibase com identidade própria e permissões de banco distintas das usadas pela aplicação.

---

### Observabilidade

Logs

- Logs estruturados no CloudWatch.
- Identificação de ambiente, componente, versão, operação e correlação.
- Auditoria de mudanças sensíveis.
- Retenção de 180 dias para registros técnicos e de auditoria.
- Sem conteúdo dos convidados ou credenciais.
- Restringir a exposição operacional de metadados de recebimento, preservando a surpresa.

Métricas

- Latência e erros por interface.
- Disponibilidade e resultados das checagens.
- Quantidade de tarefas saudáveis.
- CPU, memória e concorrência de processamento.
- Conexões, latência e recursos do banco.
- Falhas e duração de uploads.
- Idade e volume das mensagens pendentes.
- Backlog de Outbox e DLQs.
- Tempo até travamento após o primeiro registro.
- Tempo entre confirmação e mídia pronta.
- Falhas MediaConvert e processamento de fotos.
- Encaminhamento de e-mails e falhas de entrega.
- Custos por ambiente e serviço, sem orçamento máximo ainda definido.

Tracing

- Instrumentação OpenTelemetry.
- Consulta de rastros no AWS X-Ray.
- Correlação de requisição, transação local, publicação e consumo.
- Propagação de correlação pelas mensagens.
- Política de amostragem pendente de dimensionamento.
- Confirmar retenção e recursos de cada serviço: a política de 180 dias não deve ser presumida para todos os tipos de telemetria sem configuração específica.

Dashboards e alertas

- Saúde geral de produção e homologação.
- Latência, erros e carga.
- Fila e processamento de mídias.
- Banco e recuperação.
- E-mails e integrações.
- Indisponibilidade detectada e alerta disparado em até 2 minutos.
- Três envios com erro em uma janela de 5 minutos.
- Falhas definitivas de processamento e presença de mensagens em DLQ.
- Falhas de publicação e implantação.
- Alertas por SNS com assinatura do e-mail operacional.
- Alertas de orçamento pendentes da definição financeira.

---

### Riscos arquiteturais e mitigação

#### Banco e backend incompatíveis durante implantações independentes

- **Probabilidade:** média, hipótese
- **Impacto:** indisponibilidade, erros de gravação ou dificuldade de rollback.
- **Mitigação:**
  - Versionar mudanças e documentar dependências entre releases.
  - Validar breaking changes em homologação.
  - Registrar ordem necessária de publicação quando aplicável.
- **Plano de contingência:** Diórgenes executará correção ou rollback manual pelos pipelines. Não haverá coordenação automatizada obrigatória nesta versão.

#### Rollback manual ultrapassar o objetivo de recuperação

- **Probabilidade:** média, hipótese
- **Impacto:** recuperação acima de 10 minutos.
- **Mitigação:**
  - Preservar imagens imutáveis.
  - Documentar procedimentos.
  - Ensaiar recuperação em homologação.
- **Plano de contingência:** executar o procedimento manual validado e revisar a meta ou a automação caso os testes não a sustentem.

#### Recursos compartilhados entre API e consumidores

- **Probabilidade:** média, hipótese
- **Impacto:** aumento de latência e saturação durante processamento.
- **Mitigação:**
  - Limitar concorrência.
  - Usar MediaConvert para conversão pesada.
  - Testar carga combinada de API e consumidores.
- **Plano de contingência:** aumentar capacidade ou reduzir concorrência; eventual separação futura seguirá limites de domínio.

#### Permissões incorretas em tarefas com IP público

- **Probabilidade:** média, hipótese
- **Impacto:** exposição direta da aplicação ou bypass do ponto de entrada planejado.
- **Mitigação:**
  - Permitir entrada somente a partir do ALB.
  - Não permitir entrada na tarefa Liquibase.
  - Validar regras de rede e acesso externo.
- **Plano de contingência:** corrigir regras e bloquear o caminho indevido; avaliar retorno a sub-redes privadas se necessário.

#### Consistência eventual permitir alterações após o primeiro registro

- **Probabilidade:** inerente ao modelo durante a janela
- **Impacto:** alteração de dados ou aceite de convite antes do travamento.
- **Mitigação:**
  - Outbox e consumidores idempotentes.
  - Meta de processamento em até 5 segundos.
  - Alertar atrasos e falhas.
- **Plano de contingência:** recuperar mensagens e concluir o travamento. A janela foi aceita como alteração da regra estrita do PRD.

#### URLs existentes permitirem acesso após mudança para bloqueio

- **Probabilidade:** inerente durante a validade
- **Impacto:** acesso residual a conteúdo anteriormente autorizado.
- **Mitigação:**
  - Validade de até 2 minutos.
  - Renovação somente após nova autorização.
  - Não emitir novas URLs no modo bloqueado.
- **Plano de contingência:** interromper novas autorizações e tratar incidentes de exposição. Conteúdo já entregue não pode ser recolhido.

#### Mensagens repetidas ou falhas entre sistemas produzirem efeitos duplicados

- **Probabilidade:** média, hipótese
- **Impacto:** processamento, notificações ou alterações repetidas.
- **Mitigação:**
  - Outbox local, identificadores e deduplicação.
  - Consumidores idempotentes.
  - Controle persistente de trabalhos externos.
- **Plano de contingência:** investigar pelo identificador de correlação e reprocessar seletivamente. Não assumir entrega exatamente uma vez em serviços externos.

#### Reconstrução de imagem em produção diferir da versão testada

- **Probabilidade:** média, hipótese
- **Impacto:** defeito não observado em homologação.
- **Mitigação:**
  - Fixar dependências e registrar commit e versão.
  - Manter build verificável e testes unitários obrigatórios.
  - Executar validações manuais de produção.
- **Plano de contingência:** rollback manual para imagem conhecida ou publicação corretiva.

#### Integrações reais apresentarem diferenças em relação às simulações locais

- **Probabilidade:** média, hipótese
- **Impacto:** falhas descobertas tardiamente em autenticação, mídia ou mensageria.
- **Mitigação:**
  - Manter contratos comuns.
  - Planejar testes reais em homologação.
  - Validar formatos e configurações antes do evento.
- **Plano de contingência:** corrigir adaptadores e reexecutar testes; não usar simulações como evidência final de integração.

#### Metas de confiabilidade ou custo não serem atendidas

- **Probabilidade:** média, hipótese
- **Impacto:** perda de registros, indisponibilidade ou custo incompatível.
- **Mitigação:**
  - Testar persistência, recuperação e carga.
  - Medir custos por serviço.
  - Preservar originais e backups.
- **Plano de contingência:** revisar dimensionamento, arquitetura ou metas explicitamente antes do uso no casamento.

---

### ADRs e próximos passos

ADRs associados

Os itens abaixo são decisões aprovadas nesta entrevista e propostas para documentos ADR individuais no repositório `docs`. Ainda não foram criados como arquivos separados.

- ADR-001: serviço único em Clean Architecture, sem módulos, por conceito/sufixo, Controllers MVC e DbContext único; API/consumidores na mesma aplicação.
- ADR-002: frontend estático React/TypeScript no Amplify Hosting.
- ADR-003: ECS Fargate com ALB, duas tarefas mínimas em produção e escalabilidade conjunta.
- ADR-004: tarefas com IP público, entrada restrita e RDS privado, sem NAT inicial.
- ADR-005: RDS PostgreSQL, Multi-AZ em produção, schemas e contextos por módulo.
- ADR-006: Liquibase exclusivamente via GitHub Actions e tarefa Fargate.
- ADR-007: Outbox, SNS, SQS Standard, idempotência e DLQs.
- ADR-008: travamento eventual em até 5 segundos, substituindo a exigência imediata.
- ADR-009: Cognito gerenciado, sessões no PostgreSQL e cookies HttpOnly.
- ADR-010: confirmação de e-mail e recuperação por código de uma hora.
- ADR-011: upload direto, validação temporária, MediaConvert e originais preservados.
- ADR-012: CloudFront com URLs de até 2 minutos e tolerância após bloqueio.
- ADR-013: horários locais de Brasília em `timestamp without time zone`.
- ADR-014: publicação automática por branch, builds independentes e rollback manual.
- ADR-015: desenvolvimento local com Docker e simulações.
- ADR-016: somente testes unitários automáticos; demais testes acionados manualmente.

Decisões pendentes

- Domínio e subdomínios definitivos.
- Versões de .NET, React, PostgreSQL e bibliotecas.
- Classes do RDS, recursos por tarefa e limite máximo de escalabilidade.
- Critérios e métricas exatas de autoscaling.
- Parâmetros de concorrência, visibilidade, tentativas, retenção e reprocessamento das filas.
- Perfil completo de MediaConvert, incluindo bitrates e comportamento para mídia vertical.
- Bibliotecas de processamento de fotos e validação de codecs.
- Mecanismo preciso de limpeza após 24 horas.
- Contratos detalhados, endpoints e modelo físico do banco.
- Proteção, armazenamento e rotação das chaves de sessão.
- Sincronização das operações de conta e revogação de todas as sessões após alterações no Cognito.
- Adequação das telas gerenciadas ao português do Brasil, acessibilidade e regras restantes do PRD.
- Política de amostragem e retenção dos rastros.
- Orçamento e alertas financeiros.
- Escopo comprovado dos cenários de RTO/RPO.
- Procedimentos de privacidade e exclusão fora da interface.

Próximos passos

- Registrar este HLD no repositório de documentação.
- Atualizar o PRD com as mudanças explicitamente aprovadas, preservando o histórico.
- Criar os ADRs individuais.
- Elaborar guidelines e detalhamentos necessários de interfaces, dados e estados.
- Organizar seis componentes: frontend, backend, docs, tests, liquibase e infrastructure, conforme o adendo de organização.
- Usar branches `dev`, `hlg` e `main`.
- Manter `dev` sem ambiente implantado; `hlg` publica em homologação e `main` em produção.
- Exigir build e testes unitários para merge nos projetos; manter validações próprias no Liquibase.
- Publicar pelo GitHub Actions com OIDC, sem aprovação adicional após merge.
- Gerar nova imagem a cada execução do pipeline de publicação.
- Manter versões independentes por repositório, MAJOR/MINOR manuais e PATCH automático compartilhado entre `hlg` e `main`.
- Reservar versões sem colisões e usar tags imutáveis no ECR.
- Manter rollback manual em todos os componentes; rollback do banco somente por workflow Liquibase.
- Preparar Docker Compose e implementações locais simuladas.
- Configurar manualmente homologação e documentar a infraestrutura.
- Disponibilizar workflows manuais de integração, Playwright, carga e performance, com seleção de ambiente.
- Validar segurança, acessibilidade, recuperação, formatos e custos em homologação.
- Não considerar as metas técnicas comprovadas antes dos testes e da resolução das pendências impeditivas.
