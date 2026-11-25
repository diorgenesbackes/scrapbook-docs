# Plano de aceite local da F08-STORY-001

Executar `python3 -B submission_stack.py` na `dev` integrada. Usar dois eventos descartáveis, dois avisos sintéticos, relógio controlado e as migrations vigentes. Repetir `python3 -B testimonial_stack.py` para a regressão de composição e página pública. [Cartão](https://trello.com/c/r9abSKt6) · [Critérios integrados da F08](../../spec.md#7-estratégia-de-testes)

| Critério | Cenário e evidência esperada |
| --- | --- |
| CA1 | Confirmar o aviso no formulário e exigir prova do evento, autoria e versão vigentes. Renovar a prova expirada mantendo o texto. |
| CA2 | Admitir na abertura inclusiva, recusar novos ingressos no fechamento e testar o limite inclusivo de conclusão. Contar escalares Unicode e verificar autoria e horários civis no banco. |
| CA3 | Conferir registro, sessão, operação e primeiro fato. Concorrer com a mesma chave e com chaves distintas. Injetar falha da Outbox para verificar rollback e sucesso da mesma tentativa após recuperação. |
| CA4 | Perder a resposta após commit e fechar a janela. Conferir a mesma chave e credencial na recuperação. Exercitar conteúdo divergente e credenciais inválidas, vencidas ou de outro evento. |
| CA5 | Enviar via teclado, conferir foco e conteúdo do recibo, tentar novamente após perda de resposta, verificar novo ingresso com janela aberta e descarte do rascunho preservado. |

Cobrir Chromium, Firefox, WebKit, Android e iPhone emulados. Medir cinco sessões Android em 4G e 200 envios concorrentes em dois eventos. Aceitar p95 da API até 500 ms, prontidão em até 3.000 ms, recibos mínimos e ausência de dados privados nos logs. [Plano de implementação](../../plan.md)
