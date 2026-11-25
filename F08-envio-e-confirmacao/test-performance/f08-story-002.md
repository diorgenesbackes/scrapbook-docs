# Ensaio de carga local da F08-STORY-002

O ensaio independente de 25/11/2026 usou k6 2.3.0 no macOS, 200 convidados em dois eventos com 100 por evento, uma foto PNG de 922.428 bytes por tentativa, API e PostgreSQL no Docker e S3/Moto versionado. Os candidatos foram `20097b0`, `c8005e7`, `068add6`, `8d9a753` e `ef51987`. Os 200 pedidos de admissão começaram em uma janela de 11,19 ms. [Plano de aceite](../test-funcional/2026-11-25_14-54-00/1-plano.md)

| Operação | Resultado |
| --- | --- |
| Admissão | 192 de 200 respostas HTTP 201. p95 de 277,70 ms. Oito conexões terminaram com EOF. |
| Autorização | 151 de 192 respostas HTTP 200. p95 de 181,80 ms. Houve 41 EOFs. |
| Transferência ao S3 emulado | Zero de 151 PUTs completos. Houve 91 timeouts, 43 erros de buffer e outros encerramentos de conexão. |

O limiar de erros falhou e o runner encerrou com código 1. Após o ensaio, o Docker deixou de responder às consultas de diagnóstico e três chamadas de remoção dos próprios contêineres atingiram o timeout. O resultado é inconclusivo para capacidade da aplicação e transferência. O próximo ensaio comparável usa a mesma concorrência e massa pela rede descartável do Docker assim que o daemon e a limpeza forem verificados.

Na integração sem carga, a foto PNG de 922.428 bytes levou 683 ms da publicação da evidência sintética até a observação do recibo. A preparação do ensaio de carga mediu 1.038 ms para a mesma operação. Essas duas observações incluem SQS e consulta e correspondem a confirmações individuais. A prontidão 4G da entrada de foto ainda requer medição dedicada. [E2E](../test-e2e/f08-story-002.md)
