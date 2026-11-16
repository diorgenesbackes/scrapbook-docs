# Git Flow do Scrap Book

[Índice](README.md) · [Princípios](principios.md) · [Liquibase](liquibase.md) · [HLD](../HLD.md)

Adaptação do fluxo do Expenses: `main`, `hlg`, `dev`, `feature/*`, `task/*`, `merge/*` e `fix/*`, com promoção seletiva por história. A organização atual possui seis componentes, incluindo infraestrutura por definição do responsável em 2026-11-11. A política de publicação/versionamento segue o HLD.

**Status:** os seis componentes possuem repositórios independentes, com branch padrão `main` e remoto `origin`. A promoção seletiva abaixo define o processo-alvo; presença de branches, proteções, checks e pipelines precisa de evidência no componente correspondente. Os exemplos não comprovam configuração ou publicação. Revisão de planejamento e segurança: **2026-11-16**.

## 1. Modelo adotado

Cada história nasce de `main` e é integrada separadamente em `dev`, `hlg` e `main`:

```text
main → feature/123 → task/123-t01
                    task/123-t02

task/123-t01 e task/123-t02 → feature/123
feature/123 → dev → validação local do resultado integrado
feature/123 → hlg → publicação em homologação
feature/123 → main → publicação em produção

main → fix/789 → main
       fix/789 → hlg e dev
```

As setas após a criação representam PRs. Não promover o conteúdo completo de uma branch de ambiente para outra. `dev` não tem ambiente implantado; sua validação ocorre por build/unitários e testes locais acionados conforme o impacto.

Não adicionar `develop` ou `release/*` por herança de outro modelo. `scrapbook-infrastructure` é a área explicitamente solicitada para configuração operacional; sua criação não altera a decisão de configurar a infraestrutura manualmente.

## 2. Branches e nomenclatura

| Branch | Origem | Finalidade |
| --- | --- | --- |
| `main` | Base estável inicial | Produção; recebe história homologada ou hotfix. |
| `hlg` | Base estável inicial | Seleção de histórias em homologação. |
| `dev` | Base estável inicial | Integração sem deploy. |
| `feature/<id-historia>` | `main` atualizada | Entrega completa promovida aos três destinos. |
| `task/<id-historia>-tNN` | Sua feature | Item de checklist, quando precisar de branch própria; integrado somente à história. |
| `merge/<origem>-into-<destino>` | Destino | Resolver conflito de uma integração específica. |
| `fix/<id-historia>` | `main` alinhada à produção | História de correção urgente propagada a main, hlg e dev. |

Usar o ID da história no Trello, único e rastreável. T01/T02 identificam itens dentro do checklist daquela história; não são IDs de cards de tarefa. Substituir barras por hífens no nome da branch `merge/*`. História pequena pode trabalhar diretamente na feature; branch task é opcional e não justifica criar card separado.

História transversal deve identificar PRs e dependências em cada repositório: `scrapbook-ui`, `scrapbook-service`, `scrapbook-docs`, `scrapbook-tests`, `scrapbook-liquibase` e `scrapbook-infrastructure`.

### Histórias entregáveis e tarefas em checklist

O planejamento usa exclusivamente o quadro [Scrap Book no Trello](https://trello.com/b/6QX5TIHS/scrap-book). Cada história é um card com **etiqueta verde** e descreve um resultado observável para o usuário, seus critérios de aceite, limites, dependências e forma de validação. Deve ser **testável, revisável e entregável**: permitir verificar o comportamento, revisar um conjunto de mudanças compreensível e concluir uma entrega útil com as dependências declaradas. Recortar por resultado; não criar histórias isoladas de tabela, endpoint e tela quando só o conjunto permite validar a capacidade.

Subtarefas são **somente itens pequenos de checklist dentro da história**, sem cards próprios. O título deve começar por ação e explicar o objeto/resultado, permitindo entender o trabalho sem abrir outra descrição. Evitar títulos como “backend”, “ajustes” e “testes”. Se não couber em uma ação pequena e verificável, dividir o item; não impor tempo arbitrário como substituto de escopo claro.

Exemplo ilustrativo de checklist, sem criar planejamento:

```markdown
- [ ] T01 — Validar os campos editáveis do evento na EventService.
- [ ] T02 — Persistir os valores autorizados no EventRepository com comparação atômica de versão.
- [ ] T03 — Cobrir alteração válida, campo travado e versão divergente nos unitários da EventService.
```

T01 é único dentro da história; `task/123-t01` identifica sua eventual branch. Registrar esse vínculo no PR e marcar o item somente após validar seu resultado. Checklist concluído não substitui aceite da história, revisão, testes pertinentes ou publicação. Seguir Todo → Doing → Test → To Deploy e os estados adicionais existentes no quadro; etiqueta verde identifica história, sem prometer aprovação ou deploy.

## 3. Integrações e restrições

| Origem → destino | Condição |
| --- | --- |
| `task/*` → sua `feature/*` | Revisão e checks da subtarefa. |
| `feature/*` → `dev` | História pronta para integração local. |
| `feature/*` → `hlg` | Conteúdo integrado em dev, evidências e dependências identificadas. |
| `feature/*` → `main` | Aceite de homologação e candidato conferido. |
| `fix/*` → `main`, `hlg`, `dev` | Correção testada, com recuperação e propagação rastreáveis. |
| `main` → ambientes/features ativas | Sincronização controlada da base estável. |
| `merge/*` → destino indicado | Apenas sua integração específica. |

**Não integrar:** `dev → hlg`, `hlg → main`, `dev → main`, ambientes → feature/fix ou task → ambiente. Esses caminhos podem carregar histórias sem aprovação.

Merges em `hlg/main`, inclusive sincronizações e reversões, disparam publicação conforme a política do repositório. Avaliar o efeito antes de integrar; não tratar sincronização como operação sem deploy.

## 4. Desenvolvimento e promoção

Exemplos com árvore limpa e alterações locais preservadas:

```bash
git status
git fetch origin
git switch main
git pull --ff-only origin main
git switch -c feature/123
git push -u origin feature/123
```

Subtarefa após a feature existir:

```bash
git fetch origin
git switch -c task/123-t01 origin/feature/123
git push -u origin task/123-t01
```

1. Integrar subtarefas à feature com resultado e validação.
2. Abrir feature → dev e validar o resultado integrado localmente, sem presumir ambiente remoto.
3. Abrir a mesma feature → hlg; o merge publica automaticamente em homologação.
4. Acionar testes não unitários pertinentes e registrar revisão, versão e resultados.
5. Corrigir na feature/task, promover novamente e renovar evidências afetadas.
6. Conferir candidato main atual + história e dependências, registrar recuperação e abrir feature → main.

Aceite se vincula ao conteúdo e artefatos avaliados. Commit posterior ou resolução de conflito exige revisão e revalidação do impacto.

Homologação pode conter outras histórias. Validar o candidato de produção isoladamente quando os conjuntos diferirem, sem impor coordenação automatizada de releases que o HLD não prevê. Se main mudar, conferir novamente o candidato.

## 5. Dependências entre histórias e repositórios

B depende de A: registrar nos dois PRs e promover A primeiro ao destino, ou tratar A+B como entrega conjunta explicitamente validada. Não trazer dev inteira para resolver a dependência.

Para mudanças de UI/API/banco, registrar:

- Versões e contratos compatíveis.
- Changesets e configuração necessários.
- Ordem de publicação, quando houver dependência.
- Comportamento durante coexistência de versões.
- Recuperação se apenas um componente publicar.

As releases são independentes e **não haverá coordenador automatizado obrigatório**. Mudança incompatível pode causar indisponibilidade entre deploys; analisar e ensaiar a sequência em homologação. Preferir evolução compatível quando viável.

## 6. Merge e conflitos

Padrão herdado: merge commit, equivalente a `--no-ff`, preservando ancestralidade nas promoções. Não usar squash/rebase merge para a mesma feature em múltiplos destinos.

Não reescrever branches permanentes ou já compartilhadas/promovidas por rebase/force push. Não exigir histórico linear incompatível com merge commits.

Resolver conflito a partir do destino:

```bash
git fetch origin
git switch -c merge/feature-123-into-dev origin/dev
git merge --no-ff --no-commit origin/feature/123
```

Revisar arquivos, executar checks e abrir PR somente para dev. Para abortar um merge ainda em andamento, `git merge --abort`.

Nunca promover essa branch intermediária a hlg/main: ela contém o destino original. Correção de negócio descoberta no conflito também deve entrar na feature canônica. Conflito com main deve preferencialmente ser resolvido ao atualizar a feature com main antes da validação final.

## 7. PRs, revisão e checks

Cada PR informa história, origem/destino, resultado, dependências, revisão, validação, contratos, banco e recuperação. Promoção referencia os PRs/evidências anteriores.

Modelo:

```markdown
## Mudança
Problema, comportamento resultante e história.

## Integração
Origem → destino, SHA e PRs relacionados; dependências entre repositórios.

## Validação
Build/unitários, testes manuais executados, resultados e não executados.

## Publicação e recuperação
Versões, contratos, changesets, configuração, ordem e rollback.
```

Proteger main/hlg/dev por PR, checks aplicáveis, conversas resolvidas e restrição de push/force push/exclusão. Com um mantenedor, registrar revisão própria; não configurar aprovação independente impossível de cumprir rotineiramente.

**O aceite e a decisão de publicar acontecem antes do merge. Após merge, o deploy não exige aprovação adicional.** Não configurar gate manual de ambiente em conflito com o HLD.

| Área | Automação habitual | Acionamento manual conforme impacto |
| --- | --- | --- |
| Backend | Build e unitários. | HTTP, PostgreSQL, Cognito, mensageria, mídia e segurança. |
| Frontend | Build, análise estática configurada e unitários. | Playwright, dispositivos, acessibilidade e usabilidade. |
| Liquibase | Validações próprias do changelog/artefato e banco descartável conforme o pipeline definido. | Ensaios de upgrade/rollback e compatibilidade no ambiente alvo. |
| Tests | Build/validação do projeto e unitários próprios, quando existirem. | Execução das suítes não unitárias por workflow manual com seleção de ambiente. |
| Docs | Links, exemplos e consistência documental. | Revisão das regras e evidências. |

O HLD torna automáticos somente os unitários dos projetos de aplicação; não acoplar integração/E2E/carga ao merge ou deploy. O Liquibase mantém sua validação própria. Execução manual obrigatória por requisito continua obrigatória antes do aceite correspondente.

Não anunciar comandos ou scripts inexistentes. Teste ignorado/não executado não conta como aprovado. Segredos, `.env`, credenciais Liquibase, build output e arquivos temporários não entram em commits.

### Segurança dos workflows

**DEVE:** fixar Actions e workflows reutilizados em SHA completo verificado no repositório de origem; imagens usadas pelo workflow são fixadas por digest. Atualizações dessas referências passam por revisão, checks pertinentes e pela [política de dependências](principios.md#manutenção-de-dependências). Lockfile de dependências de workflow, quando adotado com ferramenta fixada, deve registrar/verificar também origens e dependências transitivas; não substituir controle por tag mutável.

GITHUB_TOKEN recebe apenas as permissões necessárias, com padrão de leitura e elevação explícita por job. id-token: write fica restrito aos jobs de publicação que precisam de OIDC. A trust policy AWS limita repositório, branch/contexto de ambiente e audiência esperados; PR não recebe identidade de deploy. Não reutilizar a mesma permissão para testes de PR e migração/publicação.

Código, parâmetros e artefatos de PR são entrada não confiável. Jobs que os executam não recebem segredos, credenciais AWS, token de escrita ou runner com acesso privilegiado. Não fazer checkout/execução de código de PR em contexto privilegiado de pull_request_target/workflow_run. Não promover artefato vindo de execução não confiável para deploy sem vincular e verificar origem, SHA elegível e proveniência. Dados de título/body/branch de PR não entram por interpolação direta em shell; usar argumentos/variáveis com tratamento apropriado.

Revisar mudanças de workflow, origem das Actions, permissões e tratamento de segredos; não considerar mascaramento de logs uma garantia contra vazamento. As práticas seguem a [referência de uso seguro do GitHub Actions](https://docs.github.com/en/actions/reference/security/secure-use). Esses controles não adicionam aprovação depois do merge nem integração/E2E automática, preservando o processo de publicação aprovado.

## 8. Commits

Conventional Commits: `<tipo>(<escopo>): <descrição>`. Tipos iniciais: `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`, `chore`, `revert`.

```text
feat(records): confirmar conjunto de fotos de forma integral
fix(events): impedir aceite concorrente de convite substituído
docs(guidelines): definir contratos de upload
test(media): verificar reprocessamento de mensagem duplicada
ci(liquibase): publicar tarefa de migração por ambiente
```

Escopos podem identificar capacidade de negócio ou camada, sem implicar módulos de código. Usar `!`/`BREAKING CHANGE:` quando houver incompatibilidade real. Commit `fix:` não exige branch fix; esta é reservada ao hotfix de produção.

## 9. Releases e artefatos

Cada repositório possui versão independente. Não criar uma tag global do conjunto nem exigir que UI/backend/banco tenham o mesmo número.

Política aprovada:

- `MAJOR` e `MINOR` são definidos manualmente.
- `PATCH` é automático, com contador compartilhado entre hlg e main **do mesmo repositório**, sem colisões.
- Cada execução do pipeline de publicação gera novo build e nova imagem onde houver imagem, inclusive produção; não promover obrigatoriamente o mesmo binário de homologação.
- Tags de imagem ECR são imutáveis; registrar digest e proveniência.
- GitHub Actions autentica na AWS por OIDC, com permissões por repositório/ambiente.
- Frontend publica artefato estático no Amplify; backend atualiza ECS; Liquibase executa tarefa Fargate.

O mecanismo de reserva durável do PATCH deve ser definido antes do pipeline, serializando concorrência de hlg/main e tratando reexecuções. Um número reservado não pode ser reutilizado com conteúdo diferente. Registrar versão e não depender apenas de contador temporário do runner. Sugestão de tag Git: `vMAJOR.MINOR.PATCH`, imutável e local ao repositório; formalizar o mecanismo na implementação.

Build novo em produção pode diferir do testado. Fixar dependências, registrar commit, versão de ferramentas, configuração e resultado do build; executar verificações controladas pós-publicação. Não afirmar equivalência binária sem evidência.

Cada deployment deve registrar: repositório, branch, SHA, versão, run do pipeline, ambiente, artefato/digest quando aplicável, configuração pública, changesets, dependências, evidências, horário, responsável e resultado.

Merge não comprova deploy bem-sucedido. A versão em produção é a identificada no registro efetivo de publicação. Serializar deploys por componente/ambiente para evitar que uma execução antiga substitua outra nova.

## 10. Hotfix

1. Conferir artefato em produção e alinhamento de main; não incluir mudanças ainda não publicadas por acidente.
2. Criar fix a partir da base estável de main e limitar ao incidente.
3. Executar checks e testes direcionados, registrar recuperação e integrar fix → main.
4. Conferir o deploy automático e executar verificação controlada.
5. Promover a mesma fix para hlg/dev e sincronizar main onde necessário.
6. Preservar evidências e encerrar a branch após propagação.

Urgência reduz escopo, não elimina rastreabilidade. O hotfix não precisa aguardar outra história.

## 11. Liquibase em paralelo

Reservar IDs/caminhos e ordenar includes por dependência. Não editar ou renomear changeset aplicado em homologação/produção. Correção deve ser incremental, sem limpar checksums para resolver merge.

Testar a evolução a partir do estado real de produção; objetos de outra história em hlg não comprovam dependência disponível. Não aplicar migration em dev remoto, pois dev não possui ambiente implantado; usar banco local descartável.

Retirar história do código não desfaz banco. Não apagar volume ou resetar ambiente compartilhado para retirar migration. O schema do candidato deve ser conhecido e reproduzível.

## 12. Rollback e encerramento

Rollback é **manual** em todos os componentes, pelos pipelines/procedimentos documentados. Backend volta a imagem conhecida; frontend, a artefato conhecido; banco, somente por workflow Liquibase. Não automatizar rollback como pressuposto do processo.

Imagem anterior precisa ser compatível com schema/configuração atuais. Git não restaura dados nem mídia. Ensaiar recuperação para avaliar a meta de 10 minutos nos cenários cobertos.

Reversão de código usa novo commit/PR, preservando histórico. Reverter merge exige escolha consciente de parent e plano para reintegrar uma correção futura.

Itens de checklist podem ser concluídos após validar seu resultado e integrar a eventual branch task; feature somente termina após publicação e sincronização necessárias. Não excluir automaticamente feature ao primeiro merge em dev/hlg. Branch de conflito termina após seu destino específico.

## 13. Critérios de adoção

- [ ] Repositórios dos seis componentes, branches e responsáveis definidos.
- [ ] Histórias são testáveis, revisáveis e entregáveis, com etiqueta verde; tarefas ficam somente em checklists legíveis e branches task opcionais rastreáveis.
- [ ] PRs/proteções respeitam promoção seletiva e mantenedor único.
- [ ] Dev sem deploy; hlg/main publicam sem aprovação posterior ao merge.
- [ ] Checks unitários e workflows manuais estão separados conforme o HLD.
- [ ] PATCH compartilhado por repositório tem reserva durável e concorrência testada.
- [ ] Novo build por publicação, tags imutáveis e proveniência registrados.
- [ ] Actions/workflows têm referências fixadas e jobs de PR/publicação usam permissões separadas, com OIDC restrito.
- [ ] Dependências entre repositórios e recuperação de deploy parcial documentadas.
- [ ] Rollbacks manuais, inclusive banco via Liquibase, ensaiados.
