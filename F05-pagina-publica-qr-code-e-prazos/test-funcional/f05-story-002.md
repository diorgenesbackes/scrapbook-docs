# Aceite funcional local da F05-STORY-002

| Critério | Evidência e pendências |
| --- | --- |
| CA1 | HTTP e PostgreSQL conferem dois responsáveis, código idêntico, resposta mínima sem cache, mesmo `404` para conta sem vínculo e evento inexistente, `401` sem credencial válida. A interface orienta nova autenticação. A sessão real depende da F02. |
| CA2 | Unitários conferem origem canônica com alias, origens inválidas e loopback. A fixture simula divergência entre origem do build e publicação. O navegador confere cópia com anúncio e seleção manual com foco. |
| CA3 | O navegador decodifica prévia, PNG e SVG e confere 60 mm, texto e controles ocultos em mídia de impressão. A abertura física via câmera de Android e iPhone continua pendente. |
| CA4 | O Playwright confere a permanência do mesmo QR durante os três estados. HTTP com PostgreSQL confirma acesso pelos dois responsáveis e consulta sem sessão usando o código original depois de uma edição da F03. |
| CA5 | O Playwright usa teclado para entrar na tela, copiar e iniciar a impressão. Nome acessível do QR e link textual foram conferidos. Leitor de tela, domínio definitivo e papel fazem parte do aceite em homologação e da preparação do piloto. |

## Roteiro para o domínio definitivo e a tiragem

1. Em `hlg`, abrir o compartilhamento com duas contas de teste vinculadas. Acessar a UI por seu domínio e por um alias do mesmo ambiente. Comparar o endereço da tela, o copiado e os endereços lidos dos dois arquivos.
2. Imprimir um exemplar a 100%. Medir pelo menos 60 mm de lado, conferir a margem branca, o nome e o endereço. Ler o exemplar com Android e iPhone e abrir a página sem sessão. Conferir a ação de download no Safari iOS e a navegação com leitor de tela.
3. Antes da tiragem do piloto, gerar o exemplar com o domínio definitivo. Ler novamente com as duas câmeras e confirmar evento e HTTPS no endereço completo. Registrar domínio, modelo dos dispositivos e resultado no card. [Card](https://trello.com/c/ZbToV6J7) · [Aceite da F05](../../PRD.md#9-critérios-de-aceitação)
