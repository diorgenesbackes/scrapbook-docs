# F10-STORY-001 — Entrega para homologação

A primeira story prepara cada foto confirmada pela F08 e disponibiliza JPEG principal e miniatura no mesmo commit. As referências ficam privadas e o recibo é preservado. [Cartão no Trello](https://trello.com/c/3tAcFFKS) · [Especificação da F10](../spec.md)

## Integrações

| Componente | PR em dev e SHA validado | PR de promoção |
| --- | --- | --- |
| Serviço | [#17](https://github.com/diorgenesbackes/scrapbook-service/pull/17) · `e9b6fe4` | [#18](https://github.com/diorgenesbackes/scrapbook-service/pull/18) |
| Liquibase | [#7](https://github.com/diorgenesbackes/scrapbook-liquibase/pull/7) · `8c1f473` | [#8](https://github.com/diorgenesbackes/scrapbook-liquibase/pull/8) |
| Infraestrutura | [#13](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/13) · `4a2c7e1` | [#14](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/14) |
| Testes | [#16](https://github.com/diorgenesbackes/scrapbook-tests/pull/16) · `a99d9b6` | [#17](https://github.com/diorgenesbackes/scrapbook-tests/pull/17) |
| Documentação inicial | [#23](https://github.com/diorgenesbackes/scrapbook-docs/pull/23) · `616cdc7` | Promoção com as evidências de aceite integradas |

A promoção em cada repositório partiu de `hlg` atualizada e selecionou os commits da story. Os candidatos de serviço, Liquibase, infraestrutura e testes têm o mesmo conteúdo das respectivas versões em `dev` validadas em laboratório. Os resultados de promoção e publicação ficam registrados no cartão.

## Evidências e aceite

- Build sem warnings, 447 unitários do serviço, 188 unitários do frontend, 31 testes Liquibase, 17 de infraestrutura e revisão independente. [Revisão técnica](../code-review/f10-story-001.md)
- CA1 a CA4 aprovados no candidato integrado, com quatro jornadas Playwright, reteste HEIF e queda controlada entre gravação e commit. A validação defensiva confirmou leitura privada, versão fixada, remoção de localização e escopo da confirmação. [Aceite funcional local](../test-funcional/f10-story-001-dev.md) · [Validação defensiva](../test-security/f10-story-001-dev.md)
- A carga local registrou 80 recibos, 90 fotos com 180 JPEGs prontos em até 16,139 s desde o início da rajada e p95 de 1,642 s da confirmação à prontidão. A rodada válida usou o relógio monotônico da fixture para contornar o salto de relógio detectado na VM. [Ensaio de carga](../test-performance/f10-story-001-dev.md)

Em homologação, confira a aplicação do changeset, as stacks F08/F10, as permissões efetivas do processo e o isolamento de rede e filesystem. Valide a entrega SNS/SQS e a DLQ, a leitura e a escrita condicionais por versão e a integridade dos JPEGs. Habilite o worker somente após essas conferências. A flag começa desabilitada. Para pausar, desabilite a flag e preserve schema, originais, variantes e trabalhos persistidos. [Infraestrutura](../dev-infra/f10-story-001.md) · [Schema e recuperação](../dev-db/f10-story-001.md)
