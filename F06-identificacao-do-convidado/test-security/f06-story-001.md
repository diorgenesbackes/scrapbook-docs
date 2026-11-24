# Verificação defensiva da primeira identificação

Em 24/11/2026, o escopo incluiu a S17 local, a consulta pública herdada da F05 e a confirmação no navegador. A validação usou exclusivamente dados sintéticos. [Candidato e ambiente](../test-e2e/f06-story-001.md)

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| White box | O código público resolvido faz parte do resultado confirmado. A rota de prévia exige diagnóstico local em loopback. O build padrão exclui a rota e o aviso de teste do JavaScript publicado. | Aprovado |
| Grey box | A consulta confirmou dois eventos abertos distintos no PostgreSQL. Ao confirmar em um evento e trocar de contexto, o segundo pediu nome e concordância novos. A suíte conferiu zero registros gravados pela etapa e ausência de nome e versão sintéticos nos logs da stack. | Aprovado |
| Black box guiado | Um navegador anônimo viu somente o contexto previsto. O código desconhecido exibiu indisponibilidade. Durante a confirmação, a URL ficou sem parâmetros de formulário. Somente os GETs públicos esperados foram observados, e a página manteve `no-referrer` e `noindex, nofollow`. | Aprovado |

Sem achados remanescentes nos controles verificados. A F08 validará a relação do evento com a versão do aviso no envio. A aprovação de privacidade é condição da abertura ao público. [Contrato](../dev-front/f06-story-001.md)
