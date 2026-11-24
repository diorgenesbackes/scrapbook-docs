# Revisão técnica da F07-STORY-003

O escopo inclui seletores de vídeo e áudio, validação de container e metadados, ciclo de vida dos players, legenda e política de mídia. [Contrato](../dev-front/f07-story-003.md)

- Tamanho e categoria são verificados antes da leitura. A inspeção mantém a leitura de cabeçalhos limitada e confirma o tipo de faixa e a duração com o navegador. A prévia aguarda dados decodificáveis e oferece recuperação em erro ou timeout.
- A seleção anterior é preservada até a troca válida. A reprodução inicia por ação do convidado. Player e prévia interrompem o carregamento na troca, no descarte ou na saída. Apenas o arquivo original e seus metadados seguem no contrato da F08.
- A política de mídia permite URLs locais `blob:` em `media-src`. A política de imagens foi preservada. O teste do pacote do Amplify verificou o escopo das diretivas nas cinco superfícies geradas.

| Check | Resultado |
| --- | --- |
| `npm run check` | Lint, TypeScript, Vite e 96 testes Vitest passaram. |
| Restore em modo bloqueado, compilação e unitários .NET 10.0.401 | Zero erros e avisos de compilação e 125 testes aprovados. |
| `git diff --check` e build padrão | Exit 0. O aviso de desenvolvimento ficou fora do JavaScript de publicação. |

Os comandos acima não coletaram cobertura numérica. Sem achados técnicos remanescentes no escopo revisado.
