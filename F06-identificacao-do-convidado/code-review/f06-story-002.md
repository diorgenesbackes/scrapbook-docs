# Revisão técnica da F06-STORY-002

Em 24/11/2026, a revisão abrangeu o estado da S17, o armazenamento por evento, o contrato para a F08 e a suíte Playwright. [Contrato e escopo](../dev-front/f06-story-002.md)

- A chave de armazenamento é derivada do código público. O registro é reconstruído depois de validar o evento, a versão do aviso, o nome e o instante. A troca de contexto remonta o fluxo. Continuar relê o registro e exige outra ação quando o nome exibido mudou em outra aba.
- O formulário de edição começa sem concordância. A aceitação explícita substitui a identificação local. Em caso de falha de gravação, a confirmação segue em memória para a página atual com feedback.
- O rascunho tem estado independente da identificação. A prévia de diagnóstico continua limitada a loopback. O build padrão exclui do JavaScript o aviso, o campo de rascunho sintético e a chave de armazenamento da prévia.
- APIs, schema e dependências permanecem nos contratos usados anteriormente. A concordância autodeclarada será validada pela F08 no envio.

| Verificação | Resultado |
| --- | --- |
| `npm run lint` e `npm run build` | Exit 0, sem diagnósticos de lint, compilação ou bundler. |
| `npm test` | 75 testes em 16 arquivos, sem coleta numérica de cobertura. |
| Restore bloqueado pelo lockfile, build e unitários do backend | Exit 0. .NET 10.0.401, 125 testes, zero avisos ou erros no build. |
| `git diff --check` | Exit 0 no escopo revisado. |

Sem achados técnicos remanescentes no escopo revisado.
