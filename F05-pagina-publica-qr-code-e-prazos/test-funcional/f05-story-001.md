# Aceite funcional local da F05-STORY-001

As verificações foram executadas no resultado integrado em `dev` com PostgreSQL preparado por Liquibase, eventos sintéticos e relógio controlado no serviço. [Card](https://trello.com/c/bRjMjxdL)

| Critério | Resultado observado |
| --- | --- |
| CA1 | A página anônima apresentou o evento correto. A próxima consulta anônima refletiu nome, boas-vindas e horários alterados na F03 pelo mesmo código. |
| CA2 | Início às 01h abriu às 23h da véspera em Brasília. Novos envios encerraram às 12h. Às 13h a página continuou fechada, preservando apenas o marco de conclusão para a F08. |
| CA3 | As duas ações aparecem desabilitadas durante a janela, somem no fechamento e o retorno à aba consulta os dados novamente. A abertura é exibida após confirmação do servidor. |
| CA4 | Código malformado ou desconhecido apresentou “Evento indisponível.”. Uma falha temporária permitiu nova consulta pelo botão. |
| CA5 | A resposta pública preservou a lista de campos, o cache desabilitado e a rota nominal nos logs. A página direta no Nginx aplicou `X-Robots-Tag` e `Referrer-Policy`. [Segurança](../test-security/f05-story-001.md) |

Resultado local aprovado para o escopo da story. QR, composição, admissão de envio e verificação da tolerância de uma tentativa pertencem aos contratos consumidores. [Rastreabilidade](../spec.md#aceite-e-integração)
