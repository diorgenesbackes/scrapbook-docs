# Revisão técnica da prévia e do aceite

| Área | Verificação |
| --- | --- |
| API | Os controllers reutilizam as verificações de identidade e origem da F03. Ambos propagam cancelamento e mapeiam erros no handler central. |
| Banco | O lock compartilhado da conta protege as condições de autenticação na transação. Evento e convite são bloqueados na ordem também usada na emissão e prevista para a F09. |
| Concorrência | Vínculo, consumo e auditoria têm o mesmo commit. Replay verifica destinatário e vínculo atual antes de devolver o identificador. |
| Privacidade | O serviço verifica e-mail antes do lookup. O identificador do evento só é devolvido no aceite e na recuperação autorizada. A saída nunca inclui segredo, contagens ou conteúdo de lembrança. |
| Interface | A ação de aceite é explícita. O fragmento sai do endereço, a prévia e o aceite são canceláveis e o erro transitório permite recuperação. |

A primeira execução do runner encontrou uma asserção de cache excessivamente estrita. Em respostas de erro, o ASP.NET envia `no-cache,no-store`. O teste passou a conferir a diretiva `no-store` na lista. O lint .NET também apontou o retorno abstrato de um método privado de leitura de chaves. A revisão adotou o tipo concreto nesse método.

A execução de navegador encontrou uma falha na prévia do Firefox ao encaminhar a requisição entre as duas APIs da fixture. Na correção, o cliente de teste passou a construir os cabeçalhos da chamada local e preservar método e corpo. A matriz repetida passou nos cinco perfis. Os checks locais passaram em 125 unitários do serviço e 50 da interface. O build .NET apresentou zero avisos e erros, e lint, build e testes de UI passaram.
