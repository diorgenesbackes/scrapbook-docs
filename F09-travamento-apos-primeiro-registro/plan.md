# Plano de implementação da F09 — Travamento após primeiro registro

[Especificação técnica](spec.md) · [PRD](../PRD.md#f09-travamento-após-primeiro-registro-1) · [HLD](../HLD.md)

## Pré-requisitos

- Versões integradas de F03, F04 e F08, com as permissões atuais de evento e convite e o primeiro fato persistido pela F08. [Contrato do fato](../F08-envio-e-confirmacao/spec.md#3-decisões-técnicas)
- Workspace com .NET 10.0.401, Node 26.13.0, npm 11.20.0, PostgreSQL local e base de Liquibase atual. O ambiente local usa migrações, identidades sintéticas e mensageria emulada. [Ambiente local](../F01-base-da-aplicacao-e-ambientes/spec.md)
- Identificar os candidatos e as dependências já disponíveis em `dev` e `hlg`, incluindo a publicação de F08 para o aceite integrado no ambiente publicado. [Git Flow](../guidelines/git-flow.md)

## Fase 1 — Preparar publicação e transporte da primeira confirmação

**1. Contrato da primeira confirmação** — Fixar a compatibilidade com os fatos que a F08 já produz para texto e mídia e estabelecer a fronteira de dados técnicos do consumidor.

**2. Mensageria da F09** — Preparar o fluxo do fato de confirmação com recursos configuráveis para desenvolvimento e homologação, incluindo recepção e recuperação de falhas.

**3. Publicação recuperável** — Entregar o despacho dos fatos elegíveis e a retomada quando houver indisponibilidade, execução concorrente ou resultado de envio incerto.

## Fase 2 — Aplicar a trava ao evento e aos convites

**4. Consumo idempotente** — Confirmar a origem de cada fato e coordenar a aplicação sem duplicar efeitos após reentregas, concorrência ou reinício.

**5. Trava e invalidação** — Aplicar a trava estrutural junto do encerramento do convite pendente, preservando a decisão original da F08 e os vínculos existentes.

**6. Integração com F03 e F04** — Preservar as transições de abertura ainda válidas e alinhar os resultados das escritas concorrentes e da recuperação de convites já aceitos.

**7. Estado visível aos responsáveis** — Fazer a configuração e os convites refletirem a trava observada no backend, com mensagens neutras e os controles permitidos disponíveis.

## Fase 3 — Entregar recuperação e preparar a ativação

**8. Resultado operacional** — Entregar os resultados de aplicação e falha para diagnóstico e consumo posterior pela F13. Registrar os sinais necessários para acompanhar atraso, pendências e recuperação.

**9. Retomada de trabalho** — Documentar a recuperação de publicações e consumos, os limites de replay e a conciliação das ocorrências anteriores à ativação.

**10. Candidato de homologação** — Preparar dependências, ordem de ativação, recuperação do candidato e evidência necessária para o aceite do ambiente publicado. [Critérios de aceite](spec.md#7-estratégia-de-testes)
