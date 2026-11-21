# F01-STORY-004 · Ensaio da recuperação local

O laboratório usou imagens candidatas Linux amd64 e um PostgreSQL 18.6 descartável com credenciais sintéticas. Os usuários padrão `app` e `liquibase` leram seus bundles públicos embutidos sem montagem extra. O ensaio de conexão usou uma CA própria para o hostname interno do banco.

| Cenário | Observado |
| --- | --- |
| TLS | O migrador recusou o modo TLS insuficiente e o hostname incompatível. O teste com o hostname autorizado confirmou identidade no PostgreSQL e conexão TLS da API. |
| Instalação e reaplicação | A imagem criou 24 tabelas e registrou um changeset. A reaplicação preservou o histórico. |
| Prévia | `preview-empty-base` concluiu e preservou as 24 tabelas e a entrada do histórico. |
| Recusas | A reversão recusou uma URL para outro banco e uma instalação com uma linha sintética. A linha e o histórico permaneceram. |
| Reversão e reinstalação | O inventário passou de 24 para 0 e voltou a 24 tabelas. O histórico passou de 1 para 0 e voltou a 1. A vitalidade permaneceu 200 e a prontidão variou de 200 para 503 e voltou a 200. |

Os contêineres e a rede próprios foram removidos. O laboratório e a simulação cobrem a preparação da versão candidata. A retomada de artefatos ativos no ECS e Amplify e o acionamento do workflow no GitHub hospedado exigem o alvo de homologação e uma publicação conhecida. [Aceite e operação da recuperação](../delivery/f01-story-004-recuperacao.md)
