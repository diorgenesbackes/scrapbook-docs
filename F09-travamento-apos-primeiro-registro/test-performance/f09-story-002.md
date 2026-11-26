# F09-STORY-002 — Carga e drenagem da retomada

O laboratório reserva a VM Docker para cada rodada e usa PostgreSQL descartável, mensageria emulada e identidades sintéticas. Os SHAs do candidato são API `8bda7e9`, infraestrutura `b86dc79`, UI `57f4e63` e Liquibase `a9be70d`. Os perfis normal e de retomada usaram suítes `f0fc3b7`. A correção `fa79c4f` altera somente a serialização do horário das notificações sintéticas no perfil misto. [Critério da F09](../spec.md#7-estratégia-de-testes)

A latência inclui o commit PostgreSQL que grava a trava. O relógio compara o timestamp do primeiro fato com o commit da versão da linha que aplicou o efeito, o que preserva a medição conservadora dos ensaios da primeira story. O ensaio com espera controlada mede à parte o tempo da liberação à drenagem. [Medição da F09-STORY-001](f09-story-001.md)

| Perfil | Comando e escopo |
| --- | --- |
| Normal | `python3 -B event_structural_lock_stack.py --load --output <arquivo>` com 100 eventos distintos e até 20 solicitações de confirmação concorrentes. |
| Retomada | `python3 -B event_structural_lock_recovery_stack.py --drain` com 20 confirmações e backlog observado antes de liberar a espera no consumidor. |
| Misto da F08 | `python3 -B event_structural_lock_recovery_stack.py --mixed-output <diretório-novo>` com texto, foto, vídeo e áudio, total de 200 contribuições em dois eventos e verificação do commit das duas travas. |

O primeiro preparo do perfil normal parou antes de gerar carga. A máscara de criação dos arquivos deixou `appsettings.json` inacessível ao usuário da imagem no snapshot isolado. A repetição ajustou as permissões da cópia usada para build e preservou as evidências privadas.

O perfil normal aprovado registrou 100 respostas HTTP `201`, pico efetivo de 20 solicitações e 100 de 100 travas em até 5 s. A duração até o commit teve p50 de 429,558 ms, p95 de 513,797 ms e máximo de 525,226 ms. O banco confirmou 100 auditorias e 100 resultados de sucesso para a F13. A Outbox de primeiros fatos e as filas principal e DLQ estavam sem trabalho pendente ao final.

Na rodada de retomada, 20 de 20 solicitações responderam `201` e o pico de clientes foi 10. O SQS acumulou 20 entregas enquanto o consumidor aguardava, sem trava aplicada. A liberação drenou a fila em 367,403 ms, com 20 travas duráveis e 20 resultados de sucesso para a F13. As filas e a Outbox de primeiros fatos ainda pendentes terminaram vazias.

Na primeira rodada mista, 200 tentativas produziram 170 uploads e 170 notificações sintéticas. Os 50 depoimentos de texto foram confirmados e as duas travas chegaram ao commit em 416,262 ms e 793,116 ms. As 150 submissões com mídia ficaram em `receiving`. O relógio da fixture emitia nove casas fracionárias. O parser de `eventTime` aceita até sete. A correção no gerador passou a serializar em microssegundos UTC, preservando massa, concorrência, limiares e deadlines do perfil.

O reteste misto confirmou 200 de 200 recibos e 890 de 890 verificações, com pico efetivo de 200 clientes, 170 uploads, 170 notificações e 170 fatos de mídia. Foram 1.330 chamadas HTTP sem falhas. Os p95 da API foram 364,766 ms na admissão, 286,984 ms na autorização, 16,341 ms na conclusão e 4,796 ms na consulta de estado. As duas travas chegaram ao commit em 377,093 ms e 773,813 ms. As duas auditorias, deduplicações e saídas de sucesso para a F13 foram confirmadas. O k6 e o runner retornaram código `0`. [PR da fixture](https://github.com/diorgenesbackes/scrapbook-tests/pull/14)

Picos amostrados de CPU e memória. O percentual de CPU usa uma CPU lógica como referência.

| Perfil | PostgreSQL | API principal | Moto |
| --- | --- | --- | --- |
| Normal | 169,53% e 126,5 MiB | 57,09% e 134,8 MiB | 0,46% e 71,38 MiB |
| Retomada | 30,03% e 94,26 MiB | 46,44% e 98,14 MiB | 0,26% e 71,27 MiB |

O inventário retornou ao estado inicial após cada rodada. A meta de até 5 s foi atendida nos perfis normal e misto. A verificação no transporte AWS é parte do aceite de homologação.
