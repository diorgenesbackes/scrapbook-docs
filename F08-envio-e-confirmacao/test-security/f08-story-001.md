# Verificação defensiva da confirmação de texto

O recorte foi a API pública de envio, a persistência e a recuperação no navegador. A validação usou exclusivamente a stack descartável com dois eventos e dados sintéticos. [Revisão técnica](../code-review/f08-story-001.md)

| Perspectiva | Controles verificados |
| --- | --- |
| Código | HMACs usam propósitos distintos e identificador de chave. A API exige prova vinculada a evento, autoria e aviso, limita o corpo e responde com `no-store`. A chave lógica e o segredo são independentes. |
| HTTP e banco | Prova de outro evento, nome trocado e aviso anterior foram recusados. Consulta sem segredo, com segredo incorreto, com outro evento ou após expiração retornou `404`. Replay vencido não abriu nova operação. A troca de conteúdo após autenticação retornou `409` e preservou o resultado original. |
| Interface e telemetria | O recibo expôs estado e instante, e a interface manteve o rascunho e a credencial em memória. A inspeção dos logs verificou que código do evento, autoria, mensagem, chave da tentativa, segredo e prova da massa de controle foram omitidos. |

A validação cobriu o transporte local em loopback e o isolamento da tentativa de texto. A ativação do aviso oficial e as integrações de mídia terão evidências próprias. [Contrato de backend](../dev-back/f08-story-001.md) · [Aceite](../test-funcional/2026-11-25_10-24-40/2-execucao.md)
