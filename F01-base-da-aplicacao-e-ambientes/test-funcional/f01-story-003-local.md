# F01-STORY-003 · Validação local em 20/11/2026

| Área | Evidência observada |
| --- | --- |
| VPC/RDS | O inventário sintético da T01 exercitou 12 cenários de rotas, sub-redes, grupos, visibilidade e criptografia. |
| API | SDK 10.0.401, restore pelo lock, build com zero warnings e 45 testes aprovados. A imagem Linux x86-64 foi construída com o SDK nativo do laboratório. |
| UI | Node 26.13.0 e npm 11.20.0. Lint, build e 10 testes passaram. O pacote Amplify foi verificado com origem HTTPS, artefato de entrada ausente e rejeição de HTTP. |
| Liquibase | Imagem Linux x86-64 construída. Os 18 casos da baseline passaram no Docker descartável. Um caso adicional confirmou a geração esperada e a recusa de uma geração divergente. |
| Reserva de versões | Quatro testes com Git remoto descartável verificaram concorrência entre `hlg` e `main`, reuso da mesma tentativa, nova tentativa e recusa de entradas inválidas. |
| Fluxos de publicação | Treze cenários com respostas AWS simuladas passaram. Foram exercitados os três caminhos felizes, recusa da API sem migração ou com revisão vencida, grupo de destino incorreto, digest divergente, ECR mutável, falha de build, falha do migrador e falha do Amplify. |
| Verificação HTTPS | Quatro testes do verificador passaram com respostas sintéticas do shell, dos cabeçalhos e da versão publicada. Os dez cenários existentes do Playwright foram enumerados. |
| Workflows | Nove arquivos passaram na leitura YAML, na verificação de eventos e permissões e na análise de sintaxe Bash. As cópias dos scripts compartilhados são idênticas. |

Reproduza a simulação em `scrapbook-tests` apontando os argumentos para os checkouts locais da mesma história. Use o checkout efetivo do Liquibase quando estiver em worktree separado. [Simulador dos fluxos](../../../scrapbook-tests/deploy/verify_workflows.py)

```sh
python3 deploy/verify_workflows.py --api "$API_WORKTREE" --ui "$UI_WORKTREE" \
    --liquibase "$LIQUIBASE_WORKTREE"
```

## Homologação pendente

Ainda faltam a configuração efetiva dos recursos, regras de tags do GitHub e roles OIDC, a primeira publicação em `hlg` e a conferência em AWS de HTTPS, prontidão, isolamento, volume no WAF e da trilha de publicação. A API, a UI, a infraestrutura e a suíte usam a base local da F01-STORY-002, cuja integração em `origin/dev` depende da promoção separada dessa história. [Preparação da homologação](../dev-infra/f01-story-003-homologacao.md) · [Ordem de publicação](../delivery/f01-story-003-publicacao.md)
