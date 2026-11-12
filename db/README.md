# Modelagem do banco de dados

Modelagem inicial: **2026-11-11**. Scripts correspondentes: **2026-11-12**. Modelo de referência para o Scrap Book, derivado do [PRD](../PRD.md), do [HLD](../HLD.md) e dos [guidelines SQL](../guidelines/sql.md).

## Ordem de leitura

| Documento | Conteúdo |
| --- | --- |
| [Modelagem](DB-Modelagem.md) | Entidades, cardinalidades, regras, estados, transações, retenção, propostas e rastreabilidade dos 22 requisitos funcionais. |
| [Diagrama ER](DB-Diagrama.md) | Visão de negócio, relações por módulo e fluxos de consistência. |
| [Modelo físico](DB-Modelo-Fisico.md) | Dicionário completo: campos, tipos, tamanhos, nulabilidade, PKs, FKs, CHECKs, índices e invariantes agregadas. |

## Escopo e status

São **33 tabelas em quatro schemas**: Identity, Events, Records e Notifications. Dessas, 21 representam domínio/coordenação e 12 são instâncias locais de Outbox, inbox e auditoria. Todas possuem PK simples inteira `id bigint GENERATED ALWAYS AS IDENTITY`.

A entrega original de 2026-11-11 continha **somente modelagem**. Por solicitação posterior, a [base Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) materializa o modelo em 47 changesets, com SQL de avanço, rollback e testes em PostgreSQL descartável. Não há mappings EF, seeds de produto, implantação AWS ou aplicação em banco compartilhado.

As regras aprovadas no produto/arquitetura são preservadas; os detalhamentos [P-01 a P-07](DB-Modelagem.md#decisões-de-detalhamento-propostas) foram usados como parâmetros desta base. Foram acrescentados os FPS racionais do original para impedir aumento de taxa na saída. Integrações, limites operacionais e procedimentos de privacidade pendentes continuam explícitos; a validação do schema não comprova AWS, desempenho ou recuperação.

## Manutenção

Alterar juntos os três documentos quando mudar entidade, campo, relacionamento ou invariante. Preservar a separação entre confirmação de recebimento e processamento, transações por módulo, autorização de abertura e ausência de dados sensíveis na mensageria/auditoria. Atualizar os índices de documentação ao mover arquivos.
