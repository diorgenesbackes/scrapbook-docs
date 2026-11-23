# Jornada de navegador da F05-STORY-001

O resultado integrado em `dev` passou nos dez cenários da F05, dois por projeto em Chromium, Firefox, WebKit, Android emulado e iPhone emulado. O Playwright navegou diretamente para `/e/{code}` com parâmetro de compartilhamento, conferiu apresentação e teclado, simulou o relógio do dispositivo divergente, controlou a hora do serviço nas bordas e observou os botões provisórios e sua remoção. Também conferiu endereço desconhecido, formato inválido e recuperação depois de uma resposta `503` simulada. [Execução local](../../../scrapbook-tests/README.md)

A primeira execução integrada encontrou cinco falhas na suíte das lembranças. O fallback de carregamento permanecia visível com o relógio do navegador pausado. Um ciclo de correção e revisão preservou a separação de módulos e passou nos 25 cenários de administração da F03, nos cinco cenários das lembranças e na jornada pública. [Revisão da correção](../code-review/f05-story-001.md)

O perfil de navegação e o teste de carga têm relatório próprio. A matriz física, leitor de tela e resposta do hosting publicado ficam para o aceite em homologação. O download e a leitura de QR pertencem à F05-STORY-002. [Desempenho](../test-performance/f05-story-001.md) · [Card da F05-STORY-002](https://trello.com/c/ZbToV6J7)
