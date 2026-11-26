# Candidato da F08-STORY-005

O convidado pode selecionar ou gravar um depoimento em áudio e retomar o envio na mesma tentativa. A API aceita MP3, M4A, WAV, Ogg ou WebM e usa o instante comprovado de recepção da versão para aplicar o limite de 13h. O recibo aparece após a confirmação persistida e a preservação do original privado. [Cartão](https://trello.com/c/KrtTW39U) · [Spec](../spec.md) · [Revisão](../code-review/f08-story-005.md)

## Integração em dev

| Repositório | PR | SHA integrado |
| --- | --- | --- |
| Serviço | [#10](https://github.com/diorgenesbackes/scrapbook-service/pull/10) | `cb102f4a55a8aeaf7a5e73a77561407e6193152a` |
| UI | [#7](https://github.com/diorgenesbackes/scrapbook-ui/pull/7) | `e3b71a564b634f0bd49cab842def52224059a310` |
| Testes | [#8](https://github.com/diorgenesbackes/scrapbook-tests/pull/8) | `51f4ca153c9f58469604546123c8c7aecab3f6a1` |
| Documentação | [#14](https://github.com/diorgenesbackes/scrapbook-docs/pull/14) | `454142ff367e094d4e122956f43b091cb8b49bf4` |
| Infraestrutura da fixture local | [#7](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/7) | `006e860c2fb5e78b8942226398ab2bc53590ed15` |

O check do serviço verificou build, unitários e imagem Linux AMD64. O ensaio inicial de capacidade registrou 200 admissões e 105 recibos no cliente antes de timeouts. [Primeira medição](../test-performance/f08-story-005-local-inicial.md)

## Validação do candidato integrado

O aceite local verificou codecs, duração e tamanho, prazo de recebimento, versão privada íntegra, recibo e saídas da F08 para F09, F10, F11 e F13. As sete jornadas Chromium passaram, incluindo microfone e retomada. O ensaio defensivo passou nos controles de isolamento, privacidade e limpeza. [Aceite funcional](../test-funcional/2026-11-26_09-16-54/2-execucao.md) · [Navegador](../test-e2e/f08-story-005-dev.md) · [Segurança](../test-security/f08-story-005-dev.md)

Após corrigir o consumo de memória da fixture, o reteste confirmou as 200 tentativas simultâneas, 100 por evento, com zero falhas HTTP. O maior p95 das operações da API medidas foi 434,345 ms. [Carga integrada](../test-performance/f08-story-005-dev.md)

## Promoção para hlg

| Componente | PR |
| --- | --- |
| Serviço | [#11](https://github.com/diorgenesbackes/scrapbook-service/pull/11) |
| UI | [#8](https://github.com/diorgenesbackes/scrapbook-ui/pull/8) |
| Testes | [#9](https://github.com/diorgenesbackes/scrapbook-tests/pull/9) |
| Infraestrutura da fixture | [#8](https://github.com/diorgenesbackes/scrapbook-infrastructure/pull/8) |

As branches seletivas foram criadas das bases atualizadas de `hlg`. A comparação confirmou que as árvores de código correspondem aos candidatos integrados e testados. Os SHAs de integração e os resultados das publicações são registrados no cartão. [Cartão F08-STORY-005](https://trello.com/c/KrtTW39U)

## Aceite do ambiente publicado

A ativação em homologação requer a ingestão de mídia existente e `PhotoSubmissions__AudioEnabled=true` na tarefa da API. A tarefa usa `ffprobe` e `ffmpeg` já incluídos na imagem e precisa de diretório temporário gravável para a validação. A F08 reutiliza o schema e as permissões de armazenamento das histórias anteriores. O roteiro verifica seleção, gravação, retomada, bloqueio de conclusão tardia, confirmação por evento nativo de versão do S3 e acesso privado ao original, com aviso de concordância aprovado. A pessoa responsável pelo ambiente conduz esse aceite. [Contrato do serviço](../dev-back/f08-story-005.md) · [Contrato da interface](../dev-front/f08-story-005.md) · [Cenários de homologação](../spec.md#7-estratégia-de-testes)
