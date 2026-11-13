# Execução — jornadas, privacidade e performance

Executor: Codex. Branch `feature/GE-STORY-001`. Backend `6bad53c0db33a301918fd28c814b750f12baf3c9`; frontend `fd473d8c1a0e9e632eb8cb1e4366c53f0ee29f83`; suítes `f7bc7f352febf0c8fccd2a297abf98e0f53c3fb0`; infraestrutura `f2a249e79f1cf57b05ba4be9d0c173571b9c1bf2`.

## Ambiente e revisões

Apple M4 Pro, arm64, 12 CPUs lógicas, 24 GiB RAM, macOS 26.6.2. PostgreSQL 18.6 em Docker Desktop; Liquibase 4.33.0, 47 migrations imutáveis. HTTP em loopback; banco na VM local do Docker. SDK .NET 10.0.401/runtime 10.0.12; Node 26.13.0/npm 11.20.0; Python 3.13.15; Playwright 1.65.0.

Comandos finais: `dotnet restore ScrapBook.sln --locked-mode`, `dotnet test ScrapBook.sln --no-restore`, `npm run check`, `python3 run.py models`, `python3 journeys.py all` e `python3 journeys.py browser` após o ajuste final de foco. Os testes não unitários permaneceram manuais.

## Resultados aprovados localmente

- Backend: **77 unitários**, zero falhas/ignorados; frontend: **39 unitários**, lint e build aprovados.
- Probe PostgreSQL persistente: contexto scoped, Unicode, filtro de vínculo, versões dos triggers e rollback aprovados.
- Dez grupos HTTP/PostgreSQL aprovados: criação/rollback, identidade/origem, isolamento, segundo responsável, entradas, concorrência/no-op, tempo/travas, obrigações, cursor/restart e OpenAPI.
- **14 jornadas de navegador**, zero falhas/ignorados na execução final. Chromium **158.0.8127.18**, desktop 1280×720/Tokyo; WebKit **27.2**, iPhone 13 emulado 390×664/Los Angeles; ambos também em 320×740. Datas civis persistidas mantiveram dia/hora e 29/02 →28/02.
- Criação, mensagem personalizada/NULL, dois salvamentos independentes, conflito/reconciliação, flags independentes, estados vazio/401/503, paginação/refresh e saúde passaram.
- POST/PUT executados com sucesso no servidor e resposta abortada no navegador: **uma mutação cada**, rascunho preservado e resultado indeterminado, sem novo envio automático.
- Foco/labels/aria-invalid/status, teclado e ausência de overflow foram conferidos. Capturas sintéticas revisadas em `capturas/`. Isso não certifica WCAG integral nem leitor de tela.
- Falha real da conexão de runtime: **2,790ms**, 503 sanitizado. Espera inicial em Identity e depois lock de Events: **4.974,508ms**, 503 dentro do orçamento compartilhado. A versão permaneceu intacta.
- **159 linhas JSON** de Events conferidas antes da carga; UTC, rota template, contexto real e campos técnicos, sem os marcadores de nome/mensagem ou a credencial. Auditoria contém somente seis campos permitidos. MeterListener/Activities unitários confirmaram dimensões e spans. Sem exportador AWS.

A execução integrada final ocorreu **22:41:09–22:42:34 de Brasília**; o reteste de navegador após foco ocorreu **22:47:55–22:48:12**. Horários completos e resultados estão no [JSON revisado](resultado-h03.json). Os registros de fases dos cartões usam o relógio do Trello; não representam horas ativas.

## Carga H-03

Uma conta, **200 eventos**, **10 concorrentes**, três ciclos de aquecimento por worker, **60s medidos**. Cada worker atualizou um evento próprio, consultando versão válida antes do PUT. Biblioteca padrão Python; sem k6. Latência inclui chamada HTTP e leitura do resultado no cliente.

| Operação | Requisições | Erros | p95 | Meta |
| --- | ---: | ---: | ---: | ---: |
| Listagem | 43.624 | 0 | 3,146ms | ≤500ms |
| Detalhe | 43.624 | 0 | 3,007ms | ≤500ms |
| Atualização | 43.624 | 0 | 11,223ms | ≤500ms |

Total: **130.872 requisições medidas**, sem incluir aquecimento/preparação. Meta local aprovada; nenhum índice/DDL foi adicionado. Resultado não demonstra capacidade de uploads ou latência de produção.

## Falhas encontradas e correções

1. Chromium perdeu nome em uma sequência de alterações: setters com snapshots completos foram substituídos por patches e atualizações funcionais. Unitário de alterações na mesma leva de render e jornadas retestados.
2. Contagem de GET no WebKit em dev incluía o ciclo adicional/cancelado do StrictMode. Jornadas passaram a usar o artefato de produção via preview; unitários também verificam um único retry transitório e nenhum retry de 4xx/mutação.
3. EF encapsulou falha de conexão, inicialmente 500. Catálogo trata tipos/SQLSTATE reconhecidos, com reteste 503 e sem retry.
4. Acessibilidade final: erro do servidor move foco ao campo; conflito/resultado indeterminado recebem foco no status. Unitário adicional e 14 jornadas passaram novamente.
5. Lint/compilação identificaram uso incompatível de ref agregada e opção exact em getByRole da Testing Library; corrigidos, sem silenciar regras.
6. O readiness inicial do container alcançava o servidor temporário do init; passou a aguardar TCP do servidor definitivo.

O aviso NO_COLOR/FORCE_COLOR do runner não alterou os resultados. Registros intermediários falhos não foram contados como aprovação. O [ciclo HTTP](../2026-11-12_21-45-00/2-execucao.md) registra o ajuste 415 e as demais falhas.

## Limites e limpeza

Somente ambiente local e massa sintética. Emulação não substitui Android/iPhone físicos, múltiplas versões de navegador, leitores de tela ou testes com pessoas. Fronteira exata de 12h é unitária; integração usa clock real e eventos encerrados. Sem Cognito, S3, filas, Records, galeria, capa customizada, disponibilidade 99,9%, RTO/RPO ou 200 uploads concorrentes.

Containers/processos exclusivos foram encerrados e os bancos descartáveis removidos. O Compose/volume e os 47 changesets do usuário foram preservados. Branches de feature aguardam revisão/promoção; nenhum deploy ou aceite humano é declarado.

## Complemento final de contrato e snapshot travado

Backend final `7a7a446d54945a08760dc0da6b783f91de1a00b0`; testes `8a327660e5d604063bc3a610e7a1890b1b5531ad`. O patch posterior ao H-03 só documenta 500/Problem Details no OpenAPI. O JSON de carga mantém a revisão do caminho de dados medido (`6bad53c`); não se atribui benchmark novo ao patch documental.

Em **23:03:27–23:03:45 de Brasília**, `python3 journeys.py browser` repetiu os dez grupos HTTP e as **14 jornadas**, todos aprovados. Conferiu 500 no OpenAPI de cada operação; PUT estrutural direto no snapshot travado retornou 409 EVENT_LOCKED, enquanto o formulário de abertura salvou uma mudança válida nos dois navegadores. Unitários backend continuaram 77/77. As capturas versionadas foram renovadas nessa execução.

A conferência de Markdown/links relativos e a matriz 26 AC foram aprovadas. Um primeiro contador simplificado de fences sinalizou falso positivo no prompt FDD existente; a validação corrigida respeita fences e info strings Markdown, sem modificar o prompt.
