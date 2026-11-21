# F03-STORY-002 · Carga local da política

O k6 usou seis usuários virtuais, duas contas vinculadas e um evento descartável durante dez segundos. Cada iteração leu a versão e tentou alterar imediatamente o modo do mesmo evento. O perfil esperava `200` ou `409 EVENT_VERSION_CONFLICT` na escrita, com p95 abaixo de 500 ms como referência do HLD.

Foram 288 leituras e 288 tentativas de alteração, com 52 alterações aceitas, 236 conflitos esperados e nenhuma resposta inesperada. O p95 da alteração foi de **10,05 ms** e a vazão agregada foi de **56,87 requisições/s**. A regressão de criação e listagem também passou, com 160 requisições e nenhuma falha HTTP.

O resultado corresponde à versão integrada em `dev` e ao perfil em loopback. Capacidade em homologação e produção depende de medição no ambiente correspondente. [HLD](../../HLD.md#considerações-de-escalabilidade-e-disponibilidade)
