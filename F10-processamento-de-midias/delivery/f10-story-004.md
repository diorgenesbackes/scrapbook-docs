# F10-STORY-004 — Candidato de recuperação de mídias

A entrega recupera confirmações verificáveis anteriores à ativação, preserva capacidade para envios novos, restaura a proteção de replays, reabre o mesmo trabalho por ciclo e remove versões elegíveis de tentativas encerradas. Os recibos e os originais da F08 permanecem preservados. A F11 recebe o estado por arquivo e as referências privadas do perfil ativo. A F13 recebe o fato definitivo correspondente ao ciclo. [Cartão no Trello](https://trello.com/c/dLVALXU9) · [Procedimento operacional](../dev-back/f10-story-004.md)

| Componente | Pull request em `dev` | SHA integrado |
| --- | --- | --- |
| Serviço | [#23](https://github.com/diorgenesbackes/scrapbook-service/pull/23) | `3834529e86e1123a4c3119c67be5395478ea6612` |
| Liquibase | [#9](https://github.com/diorgenesbackes/scrapbook-liquibase/pull/9) | `ac0e8517af47cdf4a61b81c715b0e8544040ecde` |
| Infraestrutura | [#20](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/20) | `8f6dd8cfb4226c467c48fc63e34667a82fb4b17a` |
| Testes | [#22](https://github.com/diorgenesbackes/scrapbook-tests/pull/22) | `a62469c3711a27312c9e184ab72eedc91ff7b4ff` |
| Correção da fixture | [#23](https://github.com/diorgenesbackes/scrapbook-tests/pull/23) | `4a720663ae2f1a398ba941bb2f7c10a6cc674822` |
| Documentação inicial | [#32](https://github.com/diorgenesbackes/scrapbook-docs/pull/32) | `c6eba10e48ece6d0b54b2635539ae3eef5591223` |

## Verificações locais

- Build sem avisos e 807 testes unitários do serviço sem falhas. [Revisão técnica](../code-review/f10-story-004.md)
- 36 testes Liquibase passaram na primeira revisão. Os cinco cenários de recuperação passaram na última revisão, incluindo o isolamento dos grants e o rollback preservando evidências. As 23 verificações de infraestrutura também passaram.
- 10 jornadas Chromium. Cenários de histórico, redrive, reabertura, limpeza, estados da F11 e ciclos F13 passaram. [E2E](../test-e2e/f10-story-004-dev.md) · [Aceite funcional](../test-funcional/2026-11-28_13-52-21/2-execucao.md)
- Três cenários defensivos de autorização, isolamento por evento, auditoria e limpeza da versão registrada. [Segurança defensiva](../test-security/f10-story-004-dev-security.md)
- As 80 fotos históricas e as 12 novas ficaram prontas. Todas as novas concluíram com histórico pendente. O p95 das novas foi de 1,064 s no perfil local, como cota superior de prontidão. [Carga local](../test-performance/f10-story-004/f10-story-004-dev.md)

## Aceite em homologação

Aplique a geração `f10-story-004-media-recovery` antes de ativar o runner. Configure o mesmo instante UTC de ativação no serviço e no runner. Use mídia sintética e confira redrive nativo das filas de consumo, a recusa de destinos fora do par, a resposta sem referências privadas e a preservação da versão pronta. Na limpeza, verifique a recusa de outra chave e versão sob a política temporária e a remoção da autorização ao final. Repita a distribuição mista de arquivos com a quota real do MediaConvert, em dois eventos e de acordo com o perfil da F10. [Critérios de aceite](../spec.md#aceite-rastreado-ao-prd)
