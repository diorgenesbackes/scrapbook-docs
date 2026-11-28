# F10-STORY-004 — Revisão técnica

Revisão independente da recuperação operacional, da seleção concorrente e da projeção por arquivo. O trabalho mantém as identidades canônicas da confirmação e do ciclo. A mudança exige o novo catálogo Liquibase e depende dos workers da F10-STORY-001 a F10-STORY-003. [Cartão](https://trello.com/c/dLVALXU9) · [Procedimento operacional](../dev-back/f10-story-004.md)

| Área | Correção verificada |
| --- | --- |
| Prioridade | O mesmo instante de ativação governa a varredura, a classificação no consumidor e a ordenação na Outbox. Uma vaga exclusiva por tipo atende confirmações novas. A vaga flexível alterna preferência para impedir a estagnação do histórico quando há envios contínuos. |
| Retenção | A proteção idempotente pode ser reconstruída após o arquivamento pela identidade persistida do trabalho e pela evidência da F08. Identidades ambíguas ou incompatíveis ficam registradas para intervenção. |
| Limpeza | A intenção persistida fixa a versão antes da exclusão. O lease protege retomadas, e a seleção preserva referências prontas de qualquer perfil. |
| Permissões | O login operacional recebe somente as colunas de recibo necessárias à prova. Roles distintas limitam ao par de filas previsto cada redrive e restringem exclusão S3 aos prefixos de artefatos. |
| Projeção | A consulta usa evento, registro e arquivo e só entrega o conjunto pronto do perfil ativo. |

O build do serviço passou sem avisos e os 805 testes unitários passaram. Os 36 testes Liquibase passaram na primeira revisão. Os cinco cenários de recuperação passaram após restringir a leitura e incluir a versão da limpeza. A suíte de infraestrutura passou 23 verificações. A autorização efetiva das políticas de filas e do S3 será conferida no ambiente alvo. [Parâmetros de homologação](../dev-back/f10-story-004.md)
