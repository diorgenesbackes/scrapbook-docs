# Entrega da F08-STORY-001

O convidado pode enviar um depoimento em texto revisado, confirmar que foi guardado e recuperar o mesmo recibo após perda de resposta. O backend preserva o primeiro fato de confirmação na Outbox. [Cartão](https://trello.com/c/r9abSKt6) · [Aceite local](../test-funcional/2026-11-25_10-24-40/2-execucao.md)

| Repositório | Candidato validado em `dev` | Candidato publicado no Git em `hlg` |
| --- | --- | --- |
| `scrapbook-service` | `b0ede23c433661e137ccc692b51ed7a6314f3d22` | `02aa3cb976d6c45b9fc5fc2f5c03becb0a3e94ed` |
| `scrapbook-ui` | `9c4a570da0f3a448a915baf74ae73c64305e842e` | `4dff92886949789c9a14fafd1062007c55200913` |
| `scrapbook-tests` | `1623a1b5d708be19b4751afd01c9ead064692cb7` | `a3f1aa4f624ffcd2a9a1dcaec22002e01ebee7b1` |
| `scrapbook-docs` | Spec, plano, revisão e evidências desta story | Mesmo conteúdo da F08. |

O merge seletivo preservou em `hlg` as árvores validadas da API, UI e testes. Passaram 262 testes unitários, 50 cenários funcionais de navegador, os ensaios HTTP e PostgreSQL e duas medições de navegador em 4G emulado. Os 200 envios concorrentes tiveram p95 de 207,29 ms e zero falhas. [Revisão](../code-review/f08-story-001.md) · [Navegador](../test-e2e/f08-story-001.md) · [Desempenho](../test-performance/f08-story-001.md) · [Segurança](../test-security/f08-story-001.md)

## Ensaio em homologação

As publicações da API e da UI falharam em **Reservar a versão**. Os jobs de publicação e o build da UI ficaram pulados. O candidato está em `hlg` e o ensaio no ambiente aguarda a publicação. [Falha na API](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/45623755939/job/139238422511) · [Falha na UI](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45623764569/job/139238451453)

Após a publicação, conferir a versão da API e da UI. A ativação requer a configuração `Submissions` pelo ambiente e um aviso vigente aprovado. Com o envio ativo, abrir dois links públicos em sessões independentes, enviar por teclado, conferir o horário de Brasília no recibo, interromper uma resposta após confirmação e recuperar o recibo da mesma tentativa. Conferir o bloqueio de novo ingresso após o fechamento e a renovação da concordância mantendo o texto. [Ativação](../dev-back/f08-story-001.md) · [Matriz de aceite](../test-funcional/2026-11-25_10-24-40/1-plano.md)
