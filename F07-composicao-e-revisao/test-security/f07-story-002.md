# Verificação defensiva da seleção de fotos

O escopo verificou os limites locais, os arquivos sintéticos, as URLs de prévia, a política de conteúdo e o isolamento do rascunho. [Candidato](../test-e2e/f07-story-002.md)

| Perspectiva | Evidência | Resultado |
| --- | --- | --- |
| White box | Assinatura e decodificação precedem a revisão. O payload exclui as URLs de apresentação. O Nginx e o pacote local do Amplify limitam `blob:` à diretiva de imagens. | Aprovado |
| Grey box | A seleção reteve o conjunto válido. Arquivo vazio, acima do limite, com assinatura incompatível ou sem prévia não integrou a saída. Trocar, remover e descartar revogaram as URLs observadas. | Aprovado |
| Black box guiado | A revisão ficou em memória. Os logs e o armazenamento de rascunho ficaram livres dos nomes e da legenda sintéticos. O navegador fez zero escritas HTTP e o banco manteve zero contribuições. | Aprovado |

A validação autoritativa do objeto recebido integra a F08. [Contrato](../dev-front/f07-story-002.md)
