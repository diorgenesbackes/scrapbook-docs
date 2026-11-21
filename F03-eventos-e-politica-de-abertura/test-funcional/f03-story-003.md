# F03-STORY-003 · Aceite funcional local

| Critério | Evidência | Resultado |
| --- | --- | --- |
| CA1 | Criação padrão e personalizada, edição e retorno à mensagem padrão, capa padrão. | Aprovado |
| CA2 | As duas contas responsáveis gravaram o mesmo evento. Uma terceira conta recebeu 404, e origem fora da permitida recebeu 403. | Aprovado |
| CA3 | Os três prazos novos foram conferidos no detalhe e no PostgreSQL. Escritas concorrentes produziram um sucesso e um conflito de versão. O código público permaneceu estável. | Aprovado |
| CA4 | A edição preservou o corte anterior em modo imediato no PostgreSQL. Unitários cobriram o corte programado, a materialização e a rejeição de janela incompatível. A trava estrutural de fixture bloqueou a edição por HTTP e apareceu no navegador. | Aprovado |

A trava foi preparada com um registro confirmado na base descartável. A aplicação da trava pela F09, a página pública da F05 e a identidade real da F02 pertencem às respectivas entregas.

[Regras da F03](../spec.md#regras-de-negócio) · [Execução ponta a ponta](../test-e2e/f03-story-003.md)
