# Revisão técnica da F05-STORY-002

| Área | Verificação |
| --- | --- |
| Serviço | A regra de vínculo fica em Application e reutiliza a consulta da F03. O contrato retorna só o código. Cancelamento e timeout têm tratamento distinto. |
| Interface | As consultas são canceladas ao sair da tela. A cópia manual seleciona um campo de texto. QR e URL mantêm a origem canônica. O link na página pública quebra em telas estreitas. |
| Distribuição | Um único módulo gera a matriz para prévia, downloads e impressão. SVG e PNG têm fundo branco e zona livre. A interface libera a URL temporária após iniciar o download. |
| Build e publicação | A origem da UI entra no build e acompanha seu hash. A fixture rejeita a divergência de origem antes da criação de um deploy. A recuperação de artefatos anteriores continua verificando o hash e as origens da evidência. |

Na revisão desse ciclo, o campo passou a usar `type="text"` com teclado de URL para garantir seleção manual por script. A validação de domínio foi alinhada entre build e frontend. O link público passou a quebrar linha em telas estreitas. Os checks locais incluíram restore bloqueado, build e formatação .NET, `npm run check` e 61 cenários simulados de publicação e recuperação.

Passaram 108 unitários no serviço e 41 na interface. O build .NET ficou com zero avisos e erros. `npm ci --offline` emitiu o aviso sobre o script opcional de `fsevents`. Lint, build e testes passaram em seguida.
