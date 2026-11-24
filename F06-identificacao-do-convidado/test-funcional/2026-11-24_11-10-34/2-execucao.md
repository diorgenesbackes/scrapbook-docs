# Aceite local da primeira identificação

Todos os critérios da F06-STORY-001 passaram no candidato integrado em `dev`. O runner saiu com código 0. O navegador usou a consulta F05 real e o aviso sintético conhecido pela suíte. [Ambiente e execução](../../test-e2e/f06-story-001.md)

| Critério | Observado | Resultado |
| --- | --- | --- |
| CA1 | Os dois links mostraram os nomes correspondentes. O código desconhecido retornou “Evento indisponível.” e o formulário ficou ausente. | Aprovado |
| CA2 | Página acessível sem sessão, aviso visível e caixa desmarcada nos dois eventos. | Aprovado |
| CA3 | As mensagens especificadas apareceram e o foco foi para o campo inválido. A confirmação permaneceu bloqueada nas tentativas inválidas. | Aprovado |
| CA4 | A confirmação foi anunciada no navegador e o segundo evento abriu vazio. O teste de componente demonstrou o contrato de saída com nome normalizado, código, versão e instante. | Aprovado no recorte |

O fluxo local usa a mesma regra para as futuras entradas de depoimento e momento. A aceitação pública depende da integração e da aprovação do aviso. [Contrato](../../dev-front/f06-story-001.md)
