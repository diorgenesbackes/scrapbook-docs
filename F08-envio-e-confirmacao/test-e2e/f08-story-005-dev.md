# F08-STORY-005 · Envio de áudio no Chromium em dev

As sete jornadas passaram em 43,2 s, com zero falhas, skips ou retentativas do Playwright. O runner HTTP, PostgreSQL, Moto e navegador terminou em 94,620 s com código zero. O ensaio usou Playwright 1.65.0 e Chromium headless 158.0.8127.18 em viewport de 320 px. [Cartão](https://trello.com/c/KrtTW39U) · [Spec](../spec.md#7-estratégia-de-testes)

| Componente | SHA em dev |
| --- | --- |
| Serviço | `cb102f4a55a8aeaf7a5e73a77561407e6193152a` |
| UI | `e3b71a564b634f0bd49cab842def52224059a310` |
| Testes | `51f4ca153c9f58469604546123c8c7aecab3f6a1` |
| Infraestrutura | `006e860c2fb5e78b8942226398ab2bc53590ed15` |
| Liquibase | `a9be70d2af45db878fc0506b2b7d2b62fb4d0355` |
| Base documental | `454142ff367e094d4e122956f43b091cb8b49bf4` |

| Critério | Resultado observado |
| --- | --- |
| CA1 e CA3 · Cinco arquivos selecionados | MP3, M4A, WAV, Ogg e WebM passaram pela revisão, envio e confirmação. Cada caso conferiu o recibo mínimo após a prova da versão armazenada. |
| CA1 e CA3 · Gravação | O microfone sintético gerou um áudio que pôde ser revisado e enviado sem legenda. O recebimento foi confirmado. |
| CA2 e CA3 · Retomada | O primeiro PUT foi interrompido. A retomada conservou a chave e a credencial. A interface exibiu transferência e validação e aguardou a prova. O teste fixou o recebimento da versão do Moto às 13h e publicou a notificação sintética depois. A interface confirmou e revogou a URL da prévia. |

```sh
TMPDIR=/private/tmp DOCKER_CONTEXT=colima-scrapbook-f08-validation \
  python3 -B audio_submission_stack.py --with-browser
```

A execução usou aviso, eventos, relógio e microfone de teste. O relatório JUnit e a saída da execução ficaram no laboratório privado. O exercício em dispositivos físicos e o recebimento do evento nativo do S3 dependem do aceite em homologação. [Aceite funcional](../test-funcional/2026-11-26_09-16-54/2-execucao.md)
