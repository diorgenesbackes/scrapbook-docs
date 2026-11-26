# Candidato da F08-STORY-004

O convidado pode enviar um vídeo de momento ou depoimento e retomar a mesma tentativa. A interface utiliza PUT para arquivos menores que 32 MiB e multipart em partes de 16 MiB a partir desse tamanho. A API valida o vídeo pelo conteúdo da versão fixada, preserva o original privado e publica o recibo depois do commit. [Cartão](https://trello.com/c/UR7dLziQ) · [Revisão](../code-review/f08-story-004.md) · [Plano de aceite](../test-funcional/2026-11-25_23-54-00/1-plano.md)

## Integração em dev

| Repositório | PR | SHA integrado |
| --- | --- | --- |
| UI | [#5](https://github.com/diorgenesbackes/scrapbook-ui/pull/5) | `b34aa71b21149f5481b199813f9b5e767c1dc3c4` |
| Liquibase | [#5](https://github.com/diorgenesbackes/scrapbook-liquibase/pull/5) | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |
| Infraestrutura | [#4](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/4) e [#5](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/5) | `e973ae14f0202df6b60004a83a934b71c4fd17a1` |
| Testes | [#5](https://github.com/diorgenesbackes/scrapbook-tests/pull/5) | `0ebcb76128e4c6ff5b5362606564979ca87263ad` |
| Serviço | [#7](https://github.com/diorgenesbackes/scrapbook-service/pull/7) | `203cc8365c326bf542aa08aa211f3dd5c7b3a9af` |

## Aceite do ambiente publicado

A aplicação de `f08-story-004-video-multipart` precede o serviço atualizado. A stack de mídia deve publicar o evento `CompleteMultipartUpload` de objetos temporários na fila confiável. O processo de serviço precisa de espaço temporário gravável para inspecionar até 500.000.000 bytes por réplica. O vídeo é habilitado separadamente em `PhotoSubmissions__VideoEnabled` após configuração da ingestão de mídia, da origem de upload da interface e da migration. [Infraestrutura](../dev-infra/f08-story-004.md)

O aceite em homologação inclui envio em ambas as categorias, perda de uma parte, retomada e confirmação após evento nativo vinculado à versão. Confere também IAM, SSE-KMS, CORS, SigV4, cópia privada, rejeição de conclusão tardia e limpeza de multiparts incompletos. [Spec](../spec.md#8-testes-e-evidências)
