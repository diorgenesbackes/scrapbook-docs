# F03-STORY-004 · Revisão de código

Foram revisados o caminho de estado protegido, o cálculo temporal compartilhado com o detalhe, o cancelamento de consultas e as fixtures do laboratório. Cada pedido consulta o vínculo e usa a política atual. A rota tem uma representação própria com quatro campos e a UI invalida a consulta anterior ao mudar de escopo.

As bases revisadas foram `ead5314` no serviço, `4ab7f04` na interface e `e7cbc50` nos testes. A compilação inicial do frontend apontou `TS2339` na captura de um campo da união discriminada. A captura passou a usar os campos já validados. No laboratório, a troca atômica do arquivo de relógio propagava brevemente `FileNotFoundException` na montagem Docker. A fixture passou a manter o arquivo existente e a sobrescrever os bytes de comprimento fixo antes da próxima consulta.

| Verificação do candidato | Resultado |
| --- | --- |
| Restauração bloqueada e build .NET | Saída 0, zero erros e warnings |
| `dotnet test ScrapBook.sln --no-build --no-restore` | 88 aprovados, zero falhas |
| `dotnet format ScrapBook.sln --verify-no-changes --no-restore --severity info` | Saída 0, sem diagnósticos |
| `npm run check` | Lint e build limpos, 18 unitários aprovados |

Sem achados remanescentes no escopo. [Backend](../dev-back/f03-story-004.md) · [Frontend](../dev-front/f03-story-004.md)
