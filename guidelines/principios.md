# Princípios comuns de desenvolvimento

[Índice](README.md) · [Backend](backend.md) · [Frontend](frontend.md) · [Mensageria e mídias](mensageria.md) · [Liquibase](liquibase.md)

## 1. Arquitetura orientada a responsabilidades

**DEVE:** manter regras de negócio independentes de HTTP, React, EF Core e SDKs AWS. Domain expressa invariantes; Application coordena casos de uso e define portas; Infrastructure implementa persistência e integrações; API traduz HTTP e compõe a aplicação.

Cada módulo tem essas responsabilidades, sem exigir um projeto por tabela. A direção das dependências protege regras de contas, eventos e registros. Uma pasta chamada Domain não demonstra isolamento se suas classes dependem de `HttpContext` ou S3.

No Scrap Book:

- React apresenta captura, revisão e visualização; não decide elegibilidade de envio nem abertura.
- Eventos e Convites decide vínculos, janela e abertura; Registros e Mídias controla recebimento e processamento.
- Notificações comunica; falhar um e-mail não mantém conteúdo bloqueado após a data.
- PostgreSQL mantém estados e controles persistentes; S3 mantém os arquivos; filas transportam referências.
- Liquibase evolui o schema; a aplicação não o cria no startup.

Comunicação síncrona interna usa contratos no processo. A consistência eventual aprovada usa Outbox e mensagens. Não acessar o `DbContext` de outro módulo por conveniência nem simular atomicidade distribuída.

## 2. Clean Code como critério de manutenção

| Regra | Aplicação prática |
| --- | --- |
| Nomes comunicam intenção | Preferir `ConfirmUpload`, `AuthorizePlayback` e `LockEvent` a `ProcessData`. |
| Responsabilidades são coesas | Separar validação de conteúdo, persistência, conversão e tradução HTTP. |
| Fluxos são explícitos | Guard clauses, transições nomeadas e estados que diferenciem recebimento de processamento. |
| Dados inválidos são difíceis de representar | Tipos, limites, invariantes e constraints complementares. |
| Comentários explicam decisões | Explicar tolerância até as 13h ou janela eventual de travamento. |
| Erros mantêm significado | Não confirmar sucesso após upload parcial ou ocultar falha de conversão. |
| Abstrações têm consumidores reais | Extrair o significado comum, sem um framework para possibilidades futuras. |
| Mudanças são compreensíveis | Separar refatorações amplas de novas regras quando não houver dependência. |

Código e identificadores em inglês; documentação e mensagens ao usuário em português do Brasil. Preservar nomes de contratos publicados e vocabulário consistente: evento, responsável, convidado, depoimento, momento, registro, arquivo e variante.

Não manter código morto comentado, credenciais, limites de negócio sem nome ou utilitários genéricos acumulando regras de módulos diferentes. Simplicidade não dispensa autorização, recuperação e idempotência exigidas pelo produto.

## 3. SOLID aplicado ao contexto

| Princípio | Decisão para o Scrap Book | Sinal de problema |
| --- | --- | --- |
| SRP | Controller traduz HTTP; caso de uso coordena; política decide acesso temporal. | Um controller autentica, converte vídeo e grava SQL. |
| OCP | Introduzir política quando existirem variantes reais de validação ou processamento. | Framework de plugins sem um segundo uso. |
| LSP | Adaptadores reais e simulados preservam semântica de prazo, erro e repetição. | Simulador que sempre aceita conteúdo ou promete entrega única. |
| ISP | Portas pequenas orientadas ao consumidor. | Leitura de eventos exige implementar conversão de vídeo. |
| DIP | Casos de uso dependem de contratos; detalhes Cognito/S3/SQS são adaptadores. | Domínio instancia cliente AWS ou resolve serviços no container. |

No React, composição e funções atendem a esses objetivos sem hierarquias de classes. No SQL, aplicar coesão e dependências explícitas, sem imitar literalmente o modelo de objetos.

## 4. Padrões de projeto e limites

| Padrão ou prática | Uso indicado | Limite |
| --- | --- | --- |
| Dependency Injection | Composição, lifetimes e substituição de adaptadores. | Sem Service Locator no negócio. |
| Repository / consultas | Operações persistentes com significado para o caso de uso. | Não duplicar todo o EF nem expor `IQueryable` para HTTP. |
| Unit of Work | Confirmar alterações e Outbox do módulo. | `DbContext` já fornece a base; abstração adicional exige motivo. |
| Adapter | Isolar Cognito, S3, SES, SQS, MediaConvert e transporte HTTP. | Sem camadas que apenas renomeiem chamadas. |
| Strategy | Validadores/processadores efetivamente distintos. | Não generalizar antes da necessidade. |
| Factory | Criação com invariantes e seleção relevante. | Construtor simples basta quando expressa a regra. |
| Decorator / middleware | Políticas transversais com ordem explícita. | Não esconder transações ou efeitos de negócio. |
| Composition / custom hook | Comportamento reutilizável da interface. | Hook compartilhado não implica estado global. |
| Máquina de estados | Upload, registro, processamento, convite e abertura. | Enum e transições validadas podem bastar. |
| CQRS leve | Separar leituras e comandos com necessidades diferentes. | Não exige MediatR, event sourcing ou outro banco. |
| Outbox | Persistir intenção de publicação junto do negócio. | Publicação e consumo ainda podem repetir. |
| Deduplicação / idempotência | Evitar efeitos repetidos em HTTP e consumidores. | Estado em memória ou botão desabilitado não garante unicidade entre réplicas. |

Não introduzir microsserviços, service mesh, event sourcing, Redis ou coordenador de transações distribuídas nesta versão. Os workers dos módulos permanecem na aplicação do backend, conforme o HLD.

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

## 6. Registro de decisão

Registrar ADR quando houver mudança de limite de módulo, consistência, autenticação, contrato transversal, persistência, processamento ou implantação. Decisões pequenas podem ficar na FDD e no PR.

Usar futuramente `adr/ADR-<numero>-<assunto>.md`, preservando a numeração ADR-001 a ADR-016 já enumerada no HLD:

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
- [ ] Contratos HTTP/mensagens, limites, erros e dados sensíveis revisados.
- [ ] Autorização por evento/data e proteção da surpresa verificadas.
- [ ] Repetição, concorrência e falhas parciais tratadas quando aplicáveis.
- [ ] Migração, compatibilidade e recuperação documentadas.
- [ ] Checks disponíveis executados; testes manuais e não executados identificados.
- [ ] Evidências e documentação correspondem à revisão entregue.

Não definir percentual de cobertura sem finalidade. Priorizar fronteiras temporais, isolamento de eventos, conjuntos indivisíveis, sessão revogada, respostas perdidas e mensagens duplicadas. Unitários automáticos não substituem os testes manuais obrigatórios antes do lançamento.
