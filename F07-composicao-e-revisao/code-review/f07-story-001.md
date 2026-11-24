# Revisão técnica da F07-STORY-001

Em 24/11/2026, a revisão percorreu a rota pública, a janela da F05, o editor, a revisão, a integração local da F06 e o aviso de saída. [Contrato](../dev-front/f07-story-001.md)

- A entrada usa o código do evento e reconsulta a F05. A troca do código remonta a página, e o encerramento preserva o rascunho em memória enquanto impede a identificação necessária à próxima etapa.
- Colagem acima do limite preserva o texto fora da seleção. A validação recusa texto em branco e a prévia renderiza texto como texto. O contrato só agrega identificação do mesmo evento.
- O diálogo mantém foco no controle seguro ao abrir e retorna ao link ou botão ao permanecer. A saída confirmada libera o rascunho. O fluxo público não carrega o aviso sintético no JavaScript padrão.
- A primeira execução evidenciou diferenças no evento de colagem sintético em Firefox e uma consulta de teste antes da renderização assíncrona da F06. A revisão da correção confirmou que o navegador recebe os dados da fixture e que o teste espera o campo. As asserções de edição e isolamento foram preservadas.

| Verificação | Resultado |
| --- | --- |
| Lint, TypeScript e Vite | Exit 0, sem erros ou warnings. |
| Vitest | 82 testes em 18 arquivos, sem coleta numérica de cobertura. |
| .NET 10.0.401, restore bloqueado, build e unitários | Exit 0, zero avisos no build e 125 testes. |
| Verificação do diff | Sem erros de espaços ou alterações alheias à história. |

Sem achados técnicos remanescentes no escopo revisado. O contrato da F08 e a decisão Q08 estão descritos na implementação.
