# F05-STORY-001 · Consulta pública e janela de contribuição

`GET /api/v1/public-events/{code}` consulta por código opaco e envia `Cache-Control: no-store`. O PostgreSQL usa a restrição única vigente e projeta nome, boas-vindas, início e os três marcos da janela. A resposta usa a capa padrão e o fallback de boas-vindas da F03. O contrato foi entregue sobre o schema já aplicado. [Contrato HTTP](../spec.md#5-contratos-de-api)

O formato inválido é recusado antes da consulta. Código inválido e código ausente no banco recebem o mesmo `404 RESOURCE_NOT_FOUND`, com a correlação da requisição. O limite da leitura é de dois segundos e seu cancelamento flui até EF Core. A perda do cliente continua sendo cancelamento. Uma falha transitória recebe `503 DEPENDENCY_UNAVAILABLE`.

`EventSubmissionWindowService.Describe` recebe os marcos civis persistidos e um instante do servidor. A conversão aproveita a zona de Brasília usada na F03, incluindo offsets históricos. A resposta inclui o estado e o próximo marco de transição em UTC. A F06 usa o código da rota como chave de contexto. A F07 consulta a janela pela rota pública. A F08 deverá usar os marcos vigentes no pedido e registrar os marcos da tentativa admitida. [Integrações](../spec.md#contrato-de-janela-para-f07-e-f08)

O serviço preserva a lista de campos públicos mesmo quando o evento tem contribuições confirmadas. O sinal HTTP usa a rota nominal e correlação da fundação. [Card](https://trello.com/c/bRjMjxdL)

O pico de 200 convidados revelou que o limite padrão de 100 conexões do Npgsql podia ocupar os slots disponíveis do PostgreSQL. O serviço passou a usar 32 conexões por processo como padrão, respeita o mínimo e o máximo configurados e traduz a recusa por falta de slots da consulta pública em `503`. Em homologação, conferir o orçamento total de conexões considerando o número de réplicas e os demais consumidores do banco. [Npgsql](https://www.npgsql.org/doc/connection-string-parameters.html#pooling) · [PostgreSQL](https://www.postgresql.org/docs/18/runtime-config-connection.html)
