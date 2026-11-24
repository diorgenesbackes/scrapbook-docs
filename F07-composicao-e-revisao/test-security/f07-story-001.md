# Verificação defensiva do depoimento em texto

O escopo em 24/11/2026 incluiu composição anônima, janela da F05, identificação sintética da F06 e isolamento do rascunho. [Versões e laboratório](../test-e2e/f07-story-001.md)

| Perspectiva | Controle e evidência | Resultado |
| --- | --- | --- |
| White box | A revisão renderiza texto sem interpretação de marcação. O rascunho e o payload revisado pertencem ao código atual. O snapshot de identificação de outro código é recusado. O aviso sintético está ausente do JavaScript padrão. | Aprovado |
| Grey box | A F05 revalidou a entrada com dois códigos reais de teste e relógio controlado. O fechamento impediu iniciar outra contribuição e preservou o rascunho atual. A stack manteve zero contribuições gravadas. | Aprovado |
| Black box guiado | O texto ficou fora da URL, do armazenamento local e dos logs. O nome permaneceu apenas na identificação por evento prevista pela F06, sem aparecer na URL ou logs. O navegador não realizou escrita HTTP durante a composição e a revisão. | Aprovado |

Sem achados remanescentes nos controles exercitados. A admissão autoritativa e a validação do conteúdo recebido pertencem à F08. [Contrato](../dev-front/f07-story-001.md)
