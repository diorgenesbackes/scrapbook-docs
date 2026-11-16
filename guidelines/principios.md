# Princípios comuns de desenvolvimento

[Índice](README.md) · [Backend](backend.md) · [Frontend](frontend.md) · [Mensageria e mídias](mensageria.md) · [Liquibase](liquibase.md)

## 1. Arquitetura orientada a responsabilidades

**DEVE:** manter regras de negócio exclusivamente nas Services de Application, independentes de HTTP, React, EF Core e SDKs AWS. Domain fornece dados, contratos, interfaces/portas, exceptions próprias e utilitários sem regras de negócio; Infrastructure executa persistência e integrações; API traduz HTTP, trata exceptions globalmente e compõe a aplicação. Esta é a revisão do responsável de **2026-11-15**.

O backend é um serviço único em Clean Architecture, com diretórios por conceito e sufixo obrigatório. Não criar módulos funcionais nem um projeto por tabela. A matriz de referências é API → Application/Infrastructure/Domain, Application → Domain, Infrastructure → Domain e Domain → nenhum projeto. Application e Infrastructure não se conhecem em nenhuma direção. Domain usa somente a biblioteca base do .NET; não recebe regras de negócio por serem puras. Manter adaptadores de framework na API/Infrastructure com nome técnico explícito; ver a [distribuição detalhada](backend.md#contratos-puros-e-adaptadores-técnicos).

Services pertencem exclusivamente a Application, sem referências ASP.NET/EF. Recursos técnicos são acessados por portas de Domain; por exemplo, EventCursorService usa IEventCursorProtector, implementado por EventCursorDataProtector na API. Repositórios só consultam/persistem dados: não executam callbacks de negócio, não decidem conflitos e não disparam efeitos implicitamente. Configurações EF e DbContext continuam na Infrastructure, mesmo quando mapeiam POCOs de Domain.

No Scrap Book:

- React apresenta captura, revisão e visualização; não decide elegibilidade de envio nem abertura.
- Eventos e Convites decide vínculos, janela e abertura; Registros e Mídias controla recebimento e processamento.
- Notificações comunica; falhar um e-mail não mantém conteúdo bloqueado após a data.
- PostgreSQL mantém estados e controles persistentes; S3 mantém os arquivos; filas transportam referências.
- Liquibase evolui o schema; a aplicação não o cria no startup.

Comunicação síncrona interna usa contratos no processo. A consistência eventual aprovada usa Outbox e mensagens. Repositórios compartilham um DbContext scoped; não simular atomicidade distribuída.

## 2. Clean Code como critério de manutenção

| Regra | Aplicação prática |
| --- | --- |
| Nomes comunicam intenção | Preferir `ConfirmUpload`, `AuthorizePlayback` e `LockEvent` a `ProcessData`. |
| Responsabilidades são coesas | Service decide regras; repository persiste; adaptador mede/converte; API traduz HTTP. |
| Fluxos são explícitos | Guard clauses, transições nomeadas e estados que diferenciem recebimento de processamento. |
| Dados inválidos são difíceis de representar | Services validam limites/invariantes; tipos e constraints de integridade complementam a proteção. |
| Código não contém comentários | Métodos pequenos e nomes claros; se precisa de comentário, refatorar. Motivos ficam em documentos/ADRs. |
| Erros mantêm significado | Exceptions próprias para falhas distintas; GlobalExceptionHandler na API centraliza tradução e sanitização. |
| Abstrações têm consumidores reais | Extrair o significado comum, sem um framework para possibilidades futuras. |
| Mudanças são compreensíveis | Separar refatorações amplas de novas regras quando não houver dependência. |

Código e identificadores em inglês; documentação e mensagens ao usuário em português do Brasil. Preservar nomes de contratos publicados e vocabulário consistente: evento, responsável, convidado, depoimento, momento, registro, arquivo e variante.

Não manter código morto comentado, credenciais, limites de negócio sem nome ou utilitários genéricos acumulando responsabilidades diferentes. Simplicidade não dispensa autorização, recuperação e idempotência exigidas pelo produto.

No backend, a proibição de comentários inclui comentários de linha/bloco, documentação XML e TODO/FIXME. Preservar async/await para I/O e propagar CancellationToken até as dependências canceláveis, sem bloquear tasks ou ocultar cancelamento; ver [convenções e cancelamento](backend.md#7-async-cancelamento-e-resiliência).

## 3. SOLID aplicado ao contexto

| Princípio | Decisão para o Scrap Book | Sinal de problema |
| --- | --- | --- |
| SRP | Controller traduz HTTP; Service decide negócio; repository executa persistência. | Um repository decide acesso temporal e cancela notificações ao salvar. |
| OCP | Introduzir implementações de Service quando existirem variantes reais de negócio. | Retirar regras para Policy/Validator fora das Services ou criar plugins sem necessidade. |
| LSP | Adaptadores reais e simulados preservam semântica de prazo, erro e repetição. | Simulador que sempre aceita conteúdo ou promete entrega única. |
| ISP | Portas pequenas orientadas ao consumidor. | Leitura de eventos exige implementar conversão de vídeo. |
| DIP | Services e adaptadores dependem de contratos de Domain. | Infrastructure referencia Application ou Domain instancia cliente AWS. |

No React, composição e funções atendem a esses objetivos sem hierarquias de classes. No SQL, aplicar coesão e dependências explícitas, sem imitar literalmente o modelo de objetos.

## 4. Padrões de projeto e limites

| Padrão ou prática | Uso indicado | Limite |
| --- | --- | --- |
| Dependency Injection | Composição, lifetimes e substituição de adaptadores. | Sem Service Locator no negócio. |
| Repository / consultas | Operações diretas de leitura/escrita com parâmetros explícitos. | Sem regras, callbacks de negócio, efeitos implícitos ou `IQueryable` para HTTP. |
| Unit of Work | Confirmar alterações e Outbox da operação. | `DbContext` já fornece a base; abstração adicional exige motivo. |
| Adapter | Isolar Cognito, S3, SES, SQS, MediaConvert e transporte HTTP. | Sem camadas que apenas renomeiem chamadas. |
| Strategy | Variantes reais em Services para negócio ou em adaptadores para operação técnica. | Sem mover negócio para outras classes nem generalizar antes da necessidade. |
| Factory / builder | Montagem de dados ou construção técnica com entradas já decididas. | Seleção e invariantes de negócio permanecem na Service. |
| Decorator / middleware | Políticas transversais com ordem explícita. | Não esconder transações ou efeitos de negócio. |
| Composition / custom hook | Comportamento reutilizável da interface. | Hook compartilhado não implica estado global. |
| Máquina de estados | Services validam transições de upload, registro, processamento, convite e abertura. | Enum armazena estado; entidade/repository não decide a transição. |
| CQRS leve | Separar leituras e comandos com necessidades diferentes. | Não exige MediatR, event sourcing ou outro banco. |
| Outbox | Persistir intenção de publicação junto do negócio. | Publicação e consumo ainda podem repetir. |
| Deduplicação / idempotência | Evitar efeitos repetidos em HTTP e consumidores. | Estado em memória ou botão desabilitado não garante unicidade entre réplicas. |

Organizar o backend como um serviço simples em Clean Architecture. API e consumidores permanecem na mesma aplicação, organizados por conceito. Service mesh, event sourcing, Redis e coordenação distribuída não fazem parte desta decisão.

## 5. Contratos, dados e segurança

Antes de implementar uma funcionalidade, responder:

1. Quem pode agir, sobre qual evento e em qual período?
2. Quais campos, formatos, tamanhos, durações e estados são válidos?
3. Qual é a transação local e o comportamento diante de concorrência?
4. O que acontece se houver commit, mas a resposta ou o ACK se perder?
5. Como recuperar uma falha entre banco, S3 e um serviço externo?
6. Como a versão anterior convive com o schema e as mensagens novas?
7. Que informação precisa continuar oculta até a abertura?

Autorizar cada acesso relevante no backend. Identificador, link público, chave idempotente e identificação do convidado não concedem leitura. Convidados não acessam registros após a confirmação. Responsáveis compartilham permissões, sempre restritas aos eventos vinculados e à abertura.

A surpresa inclui conteúdo, remetentes e quantidades. Respostas, caches, miniaturas, logs, alertas e permissões habituais AWS devem respeitar esse limite. Usar dados sintéticos em exemplos e testes; não transportar arquivos ou nomes reais para investigação por padrão.

Definir timezone, unidade, ausência versus `null`, ordenação, limites e idempotência nos contratos. Datas de negócio seguem Brasília; instantes técnicos não podem ser reinterpretados silenciosamente como locais.

### Manutenção de dependências

**DEVE:** registrar versão, origem, finalidade e licença das dependências diretas, incluindo pacotes, ferramentas de build, Actions e imagens. Preservar manifests/lockfiles e considerar dependências transitivas e componentes nativos na revisão de vulnerabilidades. A fixação de versão torna o build reproduzível, mas não substitui sua manutenção.

O responsável pelo projeto deve revisar avisos de segurança antes de adotar/atualizar dependências, antes de uma publicação e, enquanto houver ambiente publicado, ao menos mensalmente. Avaliar exposição real, versão afetada, correção disponível, compatibilidade e licença; registrar a atualização ou a decisão de tratamento no planejamento. Vulnerabilidade explorável que comprometa autorização, conteúdo ou integridade impede a publicação até correção ou mitigação validada.

Exceção exige registro da dependência/aviso, análise de impacto, mitigação, responsável, prazo de revisão e condição para encerramento. Não aceitar exceção sem vencimento nem atualizar major automaticamente sem avaliar contratos e regressões. Ferramentas de análise devem ter versão e escopo documentados; não anunciar scanners ou pipelines ainda inexistentes. A validação da atualização segue os testes pertinentes e o [Git Flow](git-flow.md#segurança-dos-workflows).

### Procedimentos de privacidade

A revisão de privacidade prevista no PRD deve produzir um procedimento versionado, referenciado na FDD afetada e aprovado pelo responsável antes do lançamento público. O responsável pelo projeto mantém esse registro e coordena a revisão especializada necessária; o texto de concordância no navegador, isoladamente, não encerra essa revisão.

| Tema | Registro necessário |
| --- | --- |
| Dados e acesso | Dados coletados, finalidade, armazenamento, destinatários e identidades com acesso técnico/funcional. |
| Retenção | Prazo por categoria e destino ao fim da operação da plataforma; distinguir lembranças, temporários, sessões, idempotência, auditoria e backups. |
| Aviso e menores | Versão do aviso apresentado, evidência mínima da concordância e tratamento da participação de menores definido na revisão. Mudança do aviso explicita seus efeitos sobre concordâncias anteriores. |
| Solicitações fora da interface | Canal, responsável, verificação proporcional do solicitante, escopo dos dados, decisão, execução e resposta registrada, sem acrescentar exclusão pela interface. |
| Recuperação e cópias | Como tratar originais, variantes, versões S3, backups e restaurações para não reintroduzir dados cuja remoção foi autorizada. |

Preservar as decisões atuais de retenção das lembranças enquanto a plataforma for mantida, limpeza de incompletos/rejeitados após 24 horas e registros técnicos/auditoria por 180 dias. Exceções ou mudanças decorrentes da revisão devem ser aprovadas e rastreadas antes de execução; este guideline não inventa prazo legal nem autoriza exclusão de dados reais.

## 6. Registro de decisão

Registrar ADR quando houver mudança de limites entre camadas/responsabilidades, consistência, autenticação, contrato transversal, persistência, processamento ou implantação. Decisões pequenas podem ficar na FDD e no PR.

Usar `adr/ADR-<numero>-<assunto>.md`, preservando os registros existentes e a numeração ADR-001 a ADR-016 já enumerada no HLD:

```markdown
# ADR-<numero>: <decisão>
Status: proposta | aceita | substituída
Data: AAAA-MM-DD

## Contexto e restrições
Requisito, problema, evidências e decisões anteriores.

## Alternativas
Opções viáveis e custos.

## Decisão
Escolha e razões.

## Consequências e validação
Limites, compatibilidade, implantação, testes e condição para reconsiderar.
```

## 7. Revisão e conclusão

- [ ] Requisito e decisões aprovadas do HLD atendidos, com divergências registradas.
- [ ] Dependências, transação e fonte de verdade explícitas.
- [ ] Backend concentra regras nas Services, mantém Application/Infrastructure isoladas e não contém comentários no código.
- [ ] Exceptions específicas são tratadas pelo GlobalExceptionHandler na API; async e cancelamento percorrem as operações canceláveis.
- [ ] Contratos HTTP/mensagens, limites, erros e dados sensíveis revisados.
- [ ] Dependências, vulnerabilidades, licenças e exceções com prazo foram revisadas conforme o impacto.
- [ ] Procedimento de privacidade e efeitos sobre retenção/concordância/solicitações estão registrados quando aplicáveis.
- [ ] Autorização por evento/data e proteção da surpresa verificadas.
- [ ] Repetição, concorrência e falhas parciais tratadas quando aplicáveis.
- [ ] Migração, compatibilidade e recuperação documentadas.
- [ ] Checks disponíveis executados; testes manuais e não executados identificados.
- [ ] Evidências e documentação correspondem à revisão entregue.

Não definir percentual de cobertura sem finalidade. Priorizar fronteiras temporais, isolamento de eventos, conjuntos indivisíveis, sessão revogada, respostas perdidas e mensagens duplicadas. Unitários automáticos não substituem os testes manuais obrigatórios antes do lançamento.
