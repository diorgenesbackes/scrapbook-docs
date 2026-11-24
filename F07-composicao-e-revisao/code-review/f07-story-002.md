# Revisão técnica da F07-STORY-002

Escopo de 24/11/2026. Ingresso de momento, seleção, prévias, legenda, integração com a revisão anterior e política de imagens da UI. [Contrato de fotos](../dev-front/f07-story-002.md)

- As duas rotas validam a janela da F05. A seleção inspeciona os bytes e exige decodificação do navegador. O conjunto inválido fica fora da saída. A capacidade é verificada por categoria e a troca ocupa a posição anterior.
- As prévias usam URLs revogáveis. A troca, a remoção, o descarte, a desmontagem e a interrupção da seleção liberam os recursos. O artefato local do Amplify e a resposta Nginx permitem `blob:` na diretiva de imagens.
- A legenda usa a mesma regra de contagem e colagem do texto. A revisão guarda os `File` originais em ordem e mantém as URLs de apresentação fora do contrato. A validação autoritativa continua na F08.

| Check | Resultado |
| --- | --- |
| `npm run check` | Lint e build com exit 0. Vitest aprovou 87 testes. |
| Restore em modo bloqueado, build e unitários .NET 10.0.401 | Exit 0, zero avisos na compilação e 125 testes aprovados. |
| Empacotamento local do Amplify | Política `blob:` limitada a `img-src` nas cinco superfícies geradas. |
| `git diff --check` e build padrão | Exit 0. Aviso sintético fora do JavaScript de publicação. |

A medição de cobertura numérica ficou fora dos comandos acima. A revisão corrigiu a limpeza do erro de texto obrigatório após colagem truncada na reutilização do editor. Sem achados técnicos remanescentes no escopo revisado.
