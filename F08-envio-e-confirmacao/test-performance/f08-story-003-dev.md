# Carga integrada do conjunto de fotos

## Primeira medição

A execução local de 25/11/2026 testou 200 convidados em dois eventos, com 80 conjuntos de uma foto e 20 de dez por evento. Os 560 PUTs corresponderam a 356.588.960 bytes, com PNG de 922.428 bytes e JPEG de 351.104 bytes. A integração usou PostgreSQL e armazenamento Moto em loopback no Docker dedicado. [Entrega integrada](../delivery/f08-story-003.md)

| Operação | Sucessos | p95 |
| --- | --- | --- |
| Admissão na API | 200 de 200 | 567,73 ms |
| Autorização na API | 560 de 560 | 323,34 ms |
| Sinal de conclusão | 200 de 200 | 30,55 ms |
| Transferência ao storage emulado | 560 de 560 | 1.242,23 ms |

A admissão excedeu o limite de 500 ms. A autorização e o sinal de conclusão ficaram dentro do alvo. O limite da API exclui a transferência dos arquivos. A admissão processava cada posição em uma chamada ao banco enquanto mantinha o lock do evento. Foi aberta uma correção para reduzir as chamadas no trecho transacional. [Requisito de carga](../../../PRD.md#objetivos)

O emulador omitia o identificador de versão ao enviar sua notificação automática. A rajada mediu a API e os PUTs. A prova da versão emulada usada pelo consumidor foi informada na etapa separada de reconciliação, com o identificador devolvido no PUT. [Aceite integrado](../test-funcional/2026-11-25_22-38-00/2-execucao.md)

## Prontidão local em 4G

Cinco contextos frios por entrada usaram Chromium e o perfil 4G do teste existente. O máximo foi 2.115 ms na página pública, 2.108 ms no editor de momento e 2.091 ms no editor de depoimento com foto. Os três resultados ficaram abaixo de 3 segundos. As medições usaram a UI `a46fc57`.
