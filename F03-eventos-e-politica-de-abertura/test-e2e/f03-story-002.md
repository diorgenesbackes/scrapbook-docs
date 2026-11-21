# F03-STORY-002 · Navegação e confirmação da política

O Playwright usou um PostgreSQL descartável e identidades sintéticas. As jornadas criaram um evento em modo imediato, cancelaram e confirmaram o diálogo depois do fechamento e verificaram que a data salva ficou travada. Uma sessão alterou a versão enquanto a outra mantinha um rascunho. A interface exibiu o conflito, consultou a política vigente e preservou a data do rascunho até o novo envio.

Foram dez casos da política de abertura, exercitados em Chromium, Firefox e WebKit desktop, Android emulado e iPhone emulado. Rodaram também cinco casos de criação e dez da fundação. Os ensaios foram em loopback. [Aceite funcional](../test-funcional/f03-story-002.md)
