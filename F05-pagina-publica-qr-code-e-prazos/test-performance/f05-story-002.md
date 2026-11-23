# Desempenho local da F05-STORY-002

A primeira execução da stack descartável mediu página e QR prontos em 2.119 ms, dentro da meta de 3.000 ms. A versão anterior da página ficou pronta em 2.089 ms sob o mesmo perfil. As medidas são amostras de execuções locais separadas. O perfil usa Chromium Android emulado, cache desativado, RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s. [Medição anterior](f05-story-001.md)

O k6 obteve 600 respostas de 600, p95 de 49,99 ms e zero falhas. O ensaio usa 200 convidados em dois eventos e verifica resposta sem cache. O módulo de QR tem 7,37 kB com gzip e é solicitado depois do carregamento dos dados públicos. O perfil de 200 envios simultâneos pertence à integração com a F08. [Spec e metas](../spec.md)
