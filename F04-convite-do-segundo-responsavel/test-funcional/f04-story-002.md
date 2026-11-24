# Aceite funcional local do segundo responsável

| Critério | Evidência |
| --- | --- |
| CA1 | A prévia de conta confirmada expõe nome e validade. Dois contextos comprovam que só o aceite explícito cria vínculo, e a conta passa a acessar o detalhe e a listagem da F03. |
| CA2 | A sessão ausente recebe 401. Conta não confirmada recebe 403 antes de resolver o token. O token de outro ambiente ou inválido recebe 404. Playwright confere que o fragmento é removido. |
| CA3 | O último milissegundo anterior ao prazo permite prévia. Na borda exata, prévia e aceite recusam. Convite substituído, lotação e trava têm erros distintos. |
| CA4 | A corrida de duas contas dá um único aceite. O teste de trava vence a corrida controlada. Falha de auditoria reverte vínculo e consumo. Repetição devolve o resultado apenas à conta que aceitou e permanece vinculada. |
| CA5 | O caminho do QR até “Meus eventos” funciona pelo teclado. As respostas incluem `no-store`, e os ensaios conferem que token e conteúdo das lembranças ficam fora da saída e da telemetria. |

Usar F02 para testar login, cadastro, confirmação e retomada reais. A integração F12 cobre entrega do convite por e-mail. [E2E](../test-e2e/f04-story-002.md) · [Segurança](../test-security/f04-story-002.md)
