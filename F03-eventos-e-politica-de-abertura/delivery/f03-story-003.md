# F03-STORY-003 · Entrega do candidato

A story partiu da `dev` atualizada de serviço, interface, testes e documentação. O schema vigente já oferece as colunas de boas-vindas, prazos, concorrência e trava estrutural.

| Repositório | Commit da story | Merge validado em `dev` |
| --- | --- | --- |
| Serviço | `ec4ce1b6e46a7c81f67b60e8c909232f50dca481` | `ead5314e40cb0aef7ebe1f539ddd32ef2922739c` |
| Interface | `43d319d100ddef9fc85fad1fd994aa7706e6fde5` | `4ab7f049281d87f6e3e72c8a404ef20fc1729ecf` |
| Testes | `afeff9b0fbc5db6d353b6d4f412594e49b02c03f` | `e7cbc50d77664b976e57725b4c510353e4a322d1` |

A execução integrada aprovou HTTP e PostgreSQL, aceite funcional, segurança defensiva, 25 cenários Playwright de eventos, 10 da fundação e os cenários k6. A edição concorrente apresentou p95 de 7,51 ms.

A execução local usou a infraestrutura `adf02f0` e o catálogo Liquibase `a6c5901`. O card acompanha os SHAs de integração, promoção e o estado da publicação. [F03-STORY-003 no Trello](https://trello.com/c/hCYunB6A)

Para o aceite em homologação, conferir criação com boas-vindas personalizadas, edição com recálculo dos prazos em Brasília e recuperação após conflito entre dois responsáveis. Verificar a trava estrutural em um evento preparado para esse cenário e a preservação do código público e da política vigente.

[Revisão](../code-review/f03-story-003.md) · [Aceite local](../test-funcional/f03-story-003.md) · [Segurança](../test-security/f03-story-003.md) · [Carga](../test-performance/f03-story-003.md)
