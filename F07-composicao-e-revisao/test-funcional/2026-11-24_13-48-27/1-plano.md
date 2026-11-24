# Plano de aceite do depoimento em texto

Exercitar CA1–CA5 em Docker com dois eventos sintéticos, relógio controlado e navegador anônimo. Cada projeto Playwright usa um contexto independente. A identificação da F06 usa o aviso sintético em loopback. [Trello](https://trello.com/c/3Mt5qAFJ) · [PRD](../../../PRD.md#f07-composição-e-revisão) · [Contrato](../../dev-front/f07-story-001.md)

| Critério | Cenário e resultado esperado |
| --- | --- |
| CA1 | Antes da abertura e no fechamento, a rota direta recusa iniciar. Durante a janela, o link da F05 reconsulta e abre a composição do evento correspondente. Código desconhecido apresenta indisponibilidade. |
| CA2 | Texto em branco recebe orientação. O contador aparece aos 500 restantes. Colagem Unicode respeita 5.000 e conserva o sufixo. A edição continua possível no limite. |
| CA3 | A revisão conserva o texto durante a identificação. O contrato usa código atual, categoria depoimento, formato texto e identidade do mesmo evento. |
| CA4 | Voltar à edição conserva o texto. A saída controlada abre D01. Permanecer conserva rascunho e devolve foco. Confirmar descarte limpa o conteúdo. |
| CA5 | O encerramento conserva o rascunho já aberto e bloqueia a próxima etapa. Escrever, revisar ou confirmar identificação não cria requisição de envio. |

O mesmo laboratório executa a regressão da página pública, o ensaio 4G emulado e as verificações defensivas de URL, isolamento, armazenamento de rascunho e telemetria.
