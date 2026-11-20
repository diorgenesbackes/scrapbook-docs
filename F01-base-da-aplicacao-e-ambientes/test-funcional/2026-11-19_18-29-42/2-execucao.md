# Validação funcional da F01-STORY-002 após merge em dev

**Resultado em 19/11/2026. Aceite integral pendente.** A instalação e a recuperação locais funcionaram. Faltam dados no diagnóstico de falha do Liquibase e a matriz manual de acessibilidade e compatibilidade está parcial. [Plano](1-plano.md) · [História no Trello](https://trello.com/c/nv7cVIT2)

O ciclo usou a `dev` local da API `550872b`, UI `024c436`, infraestrutura `bdf7347`, testes `9c9a15c`, documentação `1a8e778` e Liquibase `405538d`. O ambiente foi macOS 26.6.2 ARM64, Docker 29.8.2, Compose 5.6.0, PostgreSQL 18.6 e Liquibase 4.33.0. Os três projetos Compose descartáveis usaram credenciais e volumes próprios. O cleanup encerrou os projetos ao fim dos cenários. Esta execução cobre a entrega local. [Spec da F01](../../spec.md#7-estratégia-de-testes)

## Matriz de execução

| Cenário | Resultado observado e evidência | Estado |
| --- | --- | --- |
| CA1.1 · Banco vazio | O PostgreSQL recém-criado continuou sem o schema `scrapbook` depois de iniciar a API. A vitalidade ficou em `200`, e a prontidão passou de `503` antes do Liquibase a `200` após o preparo. O runner conferiu publicação em loopback, proxy e UI em pt-BR. | Aprovado |
| CA1.2 · Ordem documentada | `create_local_env.py` gerou arquivo com modo `0600`, três senhas sintéticas hexadecimais e identidade desabilitada. O segundo comando recusou sobrescrever o arquivo. Em outra stack, PostgreSQL subiu primeiro, Liquibase preparou o banco e API e UI subiram em seguida. A pessoa operadora abriu a tela e obteve “Ambiente disponível.” no Chrome. | Aprovado |
| CA2.1 · Recuperação e persistência | Com PostgreSQL parado, o proxy devolveu `200` em `/health` e `503` em `/health/ready`. Após restaurar e recriar os contêineres, a prontidão voltou a `200`. A conta sintética e o arquivo sentinela do volume de chaves foram recuperados. | Aprovado |
| CA2.2 · Falha no preparo recorrente | O runner exercitou launcher ausente, senha divergente e falha do migrador sintético. A etapa foi identificada sem os valores de teste, a credencial existente continuou válida e a API recuperou prontidão após reinício. | Aprovado |
| CA2.3 · Recuperação do catálogo | O Liquibase recusou a presença de uma tabela sintética fora do catálogo esperado, com código `1`. O histórico conservou o mesmo changeset. Após remover somente a tabela sintética, o preparo retornou `0` e a API respondeu prontidão `200`. | Aprovado |
| CA2.3 · Diagnóstico da falha real | O preparo registrou UUID da execução, ambiente `local`, etapa `Migração Liquibase` e código `1`. A revisão do candidato e o conjunto de changesets aplicados na tentativa ficaram ausentes do diagnóstico. | Reprovado |
| CA3.1 · Configuração e identidade padrão | Unitários e gerador confirmaram identidade desabilitada por padrão. A API recusou a conexão inválida, a identidade habilitada em `Production` e o modo Compose sem o opt-in de escuta. A combinação explícita em `Development` passou. As respostas de startup preservaram o diagnóstico sanitizado. | Aprovado nos cenários executados |
| CA4.1 · Teclado e estados | Os 10 casos de navegador passaram. O axe não encontrou violações nas regras habilitadas no início. A automação confirmou foco, reflow, fallback e retentativa com uma resposta inicial `503` simulada. | Aprovado no alcance automatizado |
| CA4.2 · Recuperação real no navegador | No Chrome, `Tab` alcançou “Ir para o conteúdo” e `Enter` seguido de `Tab` alcançou “Verificar disponibilidade”. Com PostgreSQL parado, acionar o botão exibiu “Serviço temporariamente indisponível. Tente novamente em instantes.” e o foco ficou em “Tentar novamente”. Após restaurar PostgreSQL, `Enter` no botão recuperou “Ambiente disponível.” na mesma aba. Uma rota ausente mostrou “Página indisponível” e “Voltar ao início” funcionou por teclado. | Aprovado |
| CA4.3 · Matriz manual | A interação por teclado foi exercitada no Chrome desktop. O cálculo de contraste da paleta passou nas combinações medidas. A aba nativa do Safari não ficou acessível no prazo da ferramenta. Faltam a matriz completa de versões, aparelhos físicos e o anúncio de estados com leitores de tela. | Parcial |

### Divergência do diagnóstico do Liquibase

A spec da F01, seção 5, pede que o diagnóstico da falha informe execução, ambiente, revisão e changesets aplicados. No catálogo descartável, a recusa ocorreu antes de aplicar qualquer changeset novo. A consulta a `scrapbook.databasechangelog` confirmou que o mesmo changeset já instalado permaneceu antes da falha, após a recusa e após a recuperação. A saída do preparo deu a etapa e o código de saída sem indicar esse conjunto vazio nem a revisão. A recuperação funcionou, com rastreabilidade operacional incompleta. [Contrato de falhas](../../spec.md#tratamento-de-falhas) · [Script de preparo](../../../../scrapbook-infrastructure/scripts/prepare_database.py)

O cenário CA3.1 cobre o fluxo documentado. A divergência de precedência em configurações mistas de escuta do Kestrel e a classificação da interrupção no cliente HTTP seguem registradas no parecer técnico. [Segunda revisão](../../code-review/f01-story-002.md#segunda-revisão-técnica)

## Navegadores e acessibilidade

| Perfil | Versão e alcance | Evidência |
| --- | --- | --- |
| Chromium desktop | 158.0.8127.18 via Playwright, viewport de 320 px | 2 de 2 |
| Firefox desktop | Motor Firefox 159.0 via Playwright | 2 de 2 |
| WebKit desktop | Motor WebKit 27.2 via Playwright | 2 de 2 |
| Pixel 7 emulado | Chromium 158.0.8127.18 | 2 de 2 |
| iPhone 13 emulado | WebKit 27.2 | 2 de 2 |
| Google Chrome nativo no Mac | Versão instalada `157.0.8103.79`. A sessão em execução ficou sem confirmação independente de versão. | Jornada real de CA4.2 e fallback exercitados pela interface |
| Safari nativo no Mac | Versão instalada `26.6.2` | A tentativa de abrir a sessão de automação expirou |
| Edge, demais versões e aparelhos físicos | Edge não estava instalado. Faltaram interfaces para teste em Android e iPhone físicos e evidência das demais versões. | Não executados |
| Leitores de tela | A árvore de acessibilidade expôs textos e controles durante o teste nativo | O anúncio com leitor de tela real permanece não executado |

Os contrastes calculados a partir do CSS foram de `5,41:1` para a introdução, `6,91:1` para disponível, `6,59:1` para indisponível e de `10,37:1` a `14,93:1` para o texto no botão. O contorno de foco obteve `5,98:1` no fundo geral e `6,52:1` no fundo branco. Os pares medidos superaram os limiares pertinentes de `4,5:1` para texto normal e `3:1` para indicadores. [CSS avaliado](../../../../scrapbook-ui/src/styles/global.css) · [Contraste textual](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) · [Contraste não textual](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html)

## Comandos e evidências

| Verificação | Resultado |
| --- | --- |
| Restore travado, build e `dotnet test` | Código `0`, 32 aprovados, zero pulados e build sem avisos. |
| `npm run check` | Código `0`, lint, build e 10 unitários aprovados. |
| `python3 docker_stack.py --browser` | Código `0`, um cenário Compose e 10 casos Playwright aprovados. |
| Jornada em Chrome nativo com PostgreSQL real | Concluída em outra stack descartável, com retomada de prontidão após reinício do banco e navegação por teclado até o início. |
| Sondagem de catálogo Liquibase e reexecução | Sondagem `0`. O `prepare_database.py` recusou a divergência com código `1` e voltou a `0` após remover a tabela sintética. |

A matriz manual e o item T11 seguem pendentes. [Runner Compose](../../../../scrapbook-tests/docker_stack.py) · [Suíte Playwright](../../../../scrapbook-tests/playwright/foundation.spec.ts) · [História no Trello](https://trello.com/c/nv7cVIT2)
