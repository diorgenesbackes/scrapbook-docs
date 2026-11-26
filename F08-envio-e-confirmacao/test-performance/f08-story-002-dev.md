# Carga integrada do depoimento com foto

Em 25/11/2026, k6 2.3.0 no macOS enviou 200 fotos PNG de 922.428 bytes, com 100 convidados por evento. O runner stock usou loopback publicado na VM Colima, relógio da fixture alinhado ao contêiner e os SHAs integrados. As 200 admissões iniciaram em 4,756 ms. [Candidato integrado](../delivery/f08-story-002.md) · [Ensaio anterior](f08-story-002.md)

| Operação | Sucessos | p95 |
| --- | --- | --- |
| Admissão na API | 200 de 200 | 447,51 ms |
| Autorização na API | 200 de 200 | 382,11 ms |
| Transferência ao S3 emulado | 200 de 200 | 528,48 ms |

Houve zero falhas HTTP em 602 chamadas e 600 de 600 checks passaram. O banco registrou 100 admissões por evento e o storage manteve 200 versões completas. O p95 das duas operações da API ficou abaixo de 500 ms. O perfil de transferência usa notificações manuais. A confirmação foi medida separadamente em 947 ms da evidência ao recibo de uma foto. [Aceite integrado](../test-funcional/2026-11-25_22-32-28/2-execucao.md)

Este perfil registra carga de admissão, autorização e transferência da história. A prontidão foto em 4G e o tempo das variantes da F10 exigem ensaios próprios. [Especificação](../spec.md#7-estratégia-de-testes)
