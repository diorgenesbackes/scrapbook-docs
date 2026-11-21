# F01-STORY-003 · Plano de validação funcional

A candidata é o diff local de `feature/F01-STORY-003` sobre `d912641` (infraestrutura), `537e725` (API), `7dc6525` (UI), `6a05707` (suítes), `fb4cbbb` (documentação) e `3b181b1` (Liquibase). A execução local usa contêineres, portas de loopback e dados sintéticos. [História no Trello](https://trello.com/c/Ji6EyWMB) · [Spec da F01](../../spec.md#7-estratégia-de-testes) · [PRD](../../../PRD.md#9-critérios-de-aceitação)

| Referência | Cenário | Resultado esperado |
| --- | --- | --- |
| CA1 · Spec §§2, 5–7 | Exercitar a fundação real em Compose, com banco descartável | API viva antes da migração, prontidão negativa até a base existir e transição para pronta após a migração. Indisponibilidade do PostgreSQL volta a derrubar apenas a prontidão. |
| CA1 e CA3 · Spec §§3 e 6 | Executar a imagem de migração em PostgreSQL local com TLS e ligar a API publicável ao mesmo banco | O certificado com nome divergente impede a migração. O certificado válido permite aplicar o changeset, instalar 24 tabelas e confirmar a geração. A reaplicação preserva o histórico. A API passa a ficar pronta após a instalação e concede CORS à origem configurada. |
| CA2 · Spec §§2 e 7 | Exercitar a regressão local do shell, teclado e disponibilidade em cinco projetos Playwright | A UI apresenta sua estrutura e controles, o foco funciona por teclado e a interface recupera a disponibilidade após uma falha sintética. Registrar motor e emulação. |
| CA2 e CA4 · Spec §§2 e 3 | Gerar e conferir o ZIP estático real usando os scripts de publicação | O ZIP contém o shell, a CSP com origem HTTPS explícita e o manifesto com revisão e versão. A regra de fallback preserva o manifesto e os arquivos estáticos. |
| CA3 e CA4 · Spec §3 | Executar os cenários de publicação com respostas AWS simuladas e o remoto Git descartável | A publicação fica vinculada à tentativa e aos artefatos, a ativação exige a base esperada e as reservas concorrentes permanecem distintas. |
| CA1–CA4 · Spec §§2, 3 e 7 | Conferir o candidato em homologação, quando o alvo estiver identificado e implantado | Inventário e isolamento AWS, ALB e WAF, HTTPS e Amplify, execução real com OIDC, ordem inicial de deploy e evidência da versão publicada por componente. |

As asserções dos cenários TLS usam uma autoridade certificadora de laboratório e um nome resolvido somente na rede Docker. Os critérios de homologação exigem evidência do ambiente indicado na história.
