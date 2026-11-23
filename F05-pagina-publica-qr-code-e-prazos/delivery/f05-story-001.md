# F05-STORY-001 · Entrega do candidato

A página pública e a consulta de horários foram integradas em `dev`. A promoção seletiva para `hlg` partiu das branches atualizadas, preservou a ancestralidade de cada destino e levou somente os commits da story. Os resultados de código e testes em `hlg` têm a mesma árvore que a versão validada de `dev`. A F03 e o schema usados pela consulta já fazem parte dos destinos. [Card](https://trello.com/c/bRjMjxdL)

| Repositório | `dev` validado | Candidato em `hlg` |
| --- | --- | --- |
| Serviço | `3e4c3350678b06c27e9012d4ed8f012809f17984` | `b369f726bbd31b81ad5800d7288b5c0b6005086c` |
| Interface | `1ae1310ac080fa91c05c09efbeffe2f9886b2736` | `2173efe1a6d2140bd56a3985b49e256bed245e9b` |
| Testes | `0d3741da7c920b301f86799b1c6ce714867ea6b8` | `dad7eece5553969fd5562219322df589b2d3fd3c` |

O candidato local passou em 107 unitários de serviço, 26 de interface, HTTP e PostgreSQL, 10 cenários Playwright da F05 e 30 regressões de navegador da F03. A medição final em `dev` obteve 600 de 600 respostas, p95 de 62,16 ms e página pronta em 2.089 ms no perfil 4G. [Revisão](../code-review/f05-story-001.md) · [Aceite local](../test-funcional/f05-story-001.md) · [E2E](../test-e2e/f05-story-001.md) · [Segurança](../test-security/f05-story-001.md) · [Desempenho](../test-performance/f05-story-001.md)

## Publicação em homologação

Os workflows da API e da interface falharam em “Reservar a versão”. Ambos registraram “Use a série MAJOR.MINOR em RELEASE_SERIES.” O GitHub Actions pulou a publicação dos dois componentes. O aceite em homologação depende de uma execução com a série de release definida para os dois repositórios. [Workflow da API](https://github.com/diorgenesbackes/scrapbook-service/actions/runs/45224407760) · [Workflow da interface](https://github.com/diorgenesbackes/scrapbook-ui/actions/runs/45224417006) · [Política de versões](../../guidelines/git-flow.md#9-releases-e-artefatos)

## Cenários para o aceite em `hlg`

1. Confirmar os SHAs publicados. Usar eventos de teste com código conhecido de um fixture autorizado no ambiente.
2. Abrir o link sem sessão em Android, iPhone e desktop. Conferir a página, a navegação por teclado, o leitor de tela e os headers nas respostas publicadas.
3. Conferir os três estados em Brasília, inclusive a retirada das ações às 12h e a manutenção do encerramento às 13h. Verificar a mensagem provisória durante a janela e o mesmo link após uma edição permitida na F03.
4. Conferir a lista de campos públicos e a telemetria do edge. Medir a página no perfil 4G e estimar o pico de conexões com as réplicas e os demais consumidores do RDS.
