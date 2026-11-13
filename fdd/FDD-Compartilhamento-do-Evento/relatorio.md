# Relatório: PP-STORY-003

[História](https://trello.com/c/yUy0Yq2f) · [Contrato](FDD-Compartilhamento-do-Evento.md) · [Execução](../../testing/2026-11-13_15-57-13/2-execucao.md)

## Resultado

Compartilhamento implementado em feature/PP-STORY-003: consulta autorizada para os dois responsáveis, tela S15 acessível pelo detalhe, link canônico, cópia com alternativa manual, QR reutilizado em S15/S16, download PNG e material branco para impressão. Todos os formatos codificam a mesma URL configurada no servidor. O código público identifica o evento; não autoriza gestão ou mídia privada.

Antes desta feature, a stack persistente foi parada e PP-STORY-002 integrada nas mains de service/UI/tests/docs por solicitação do responsável. Merges sem conflitos, com árvores iguais às versões validadas, enviados ao origin. [Auditoria](../../testing/2026-11-13_15-57-13/integracao-PP-STORY-002.json). A pendência anterior PP-002-029, leitor de tela real, permanece incompleta. A nova história nasceu dessas mains e não foi integrada nem publicada.

**Situação:** 35 subtarefas em To Deploy; **PP-003-034 em Test/incompleta**, aguardando leitura em aparelhos reais e papel. A história permanece em Test. Não declarar aceite físico a partir de decodificação digital.

## Validação

| Verificação | Resultado |
| --- | --- |
| Backend | Build sem avisos/erros; 120 testes xUnit aprovados. |
| Frontend | Lint/build e 62 testes Vitest aprovados. |
| HTTP/PostgreSQL | Dois responsáveis autorizados, outsider negado, código público sem poder administrativo, OpenAPI, Host adulterado e ausência/origem HTTPS aprovados. |
| Compartilhamento | Dez testes Chromium/WebKit aprovados: QR exibido, captura, PNG e mídia print decodificados por jsQR, independente do gerador. |
| Regressão | 11 testes de apresentação/performance, seis de acesso público e 14 de gestão/health aprovados. Total de 41 cenários distintos incluindo compartilhamento. |
| Página em perfil 4G | Nome, mensagem, capa e QR prontos em 741/739/742/769/736ms; máximo 769ms, limite 3000ms. |
| Carga de leitura pública | 1004 eventos, dez clientes, 60s: 265946 consultas, zero erros, p95 3,43ms, máximo 37,018ms. |
| Privacidade | Snapshots das tabelas de aplicação preservados após leituras; logs amostrados sem nome/código/URL utilizável. |

Chromium 158.0.8127.18 e WebKit 27.2; composição móvel emulada, sem aparelho real. Perfil 4G customizado: download 10Mbps/upload 2Mbps decimais, latência mínima de headers 80ms, CPU 4×, 390×844, cinco contextos novos/cache desabilitado. As medições são locais, não representam rede comercial ou AWS. Gestão foi executada nos dois laboratórios de regressão e contada uma vez. Nenhum teste foi ignorado ou aprovado por retry automático.

## Matriz de aceite

- AC01: a URL de link/QR/download/print é idêntica, decodificada e aberta anonimamente no evento correto; verificação física ainda pendente.
- AC02: os dois responsáveis têm a mesma projeção mínima; terceiro recebe 404 e requisição privada sem identidade recebe 401.
- AC03: QR e link público funcionam em contexto anônimo; controles de exportação ficam na gestão.
- AC04: contraste, quatro módulos de margem e nome/link/QR verificados digitalmente; papel/aparelhos reais pendentes em PP-003-034.
- AC05: cópia só confirma após resolução; seleção manual, retry de geração, erro de PNG e orientação de impressão cobertos. Não afirmar que o papel foi impresso.

## Entrega por subtarefa

[Transições observadas em UTC](../../testing/2026-11-13_15-57-13/transicoes.json) registram o fluxo. Horários são observações do cliente após operações do conector, não horas ativas exclusivas ou timestamps nativos de mudança do Trello.

| ID | Estado | Entrega/evidência |
| --- | --- | --- |
| PP-003-001 | To Deploy | PublicEventLinkPolicy única; concatena origem confiável e código existente para todos os formatos. |
| PP-003-002 | To Deploy | PublicSiteOptions, modelo imutável e injeção; Host/Forwarded não participam da URL. |
| PP-003-003 | To Deploy | FDD registra domínio real pendente; origem local explícita e ausência segura em Production. |
| PP-003-004 | To Deploy | GET /api/v1/events/{eventId}/share retorna apenas code/name/publicUrl; público acrescenta publicUrl nulável. |
| PP-003-005 | To Deploy | QR 1024px, M, preto/branco opaco, margem quatro módulos; PNG 1280px e print 70mm. |
| PP-003-006 | To Deploy | qrcode 1.5.4 e @types/qrcode 1.5.6 MIT fixados; somente manifest/lockfile versionados. |
| PP-003-007 | To Deploy | EventSharingService reutiliza código existente e só compõe URL depois da consulta autorizada. |
| PP-003-008 | To Deploy | EventSharingRepository projeta code/name, AsNoTracking e filtro de vínculo; nenhuma gravação. |
| PP-003-009 | To Deploy | EventController, rota share, erros 400/401/404/503 e orçamento/cancelamento existentes. |
| PP-003-010 | To Deploy | OpenAPI documenta operação privada e publicUrl obrigatório/nulável na projeção pública. |
| PP-003-011 | To Deploy | Política recusa origem com caminho/credenciais/query/fragmento; HTTPS externo e HTTP só loopback explícito. |
| PP-003-012 | To Deploy | Compose/.env.example/helpers propagam origem; variável de ambiente precede arquivo e fallback da porta UI. |
| PP-003-013 | To Deploy | Link no detalhe e rota /events/:eventId/share preservam retorno ao evento. |
| PP-003-014 | To Deploy | EventSharePage com consulta autorizada, nome, link, loading e erro contextual. |
| PP-003-015 | To Deploy | Clipboard API com sucesso anunciado apenas após Promise resolvida. |
| PP-003-016 | To Deploy | Falha/ausência de Clipboard fornece input readonly focado e selecionado. |
| PP-003-017 | To Deploy | EventSharePanel/qr.ts reutilizáveis; URL validada sem substituir origem pela do navegador. |
| PP-003-018 | To Deploy | S16 anônima mostra QR/link; ausência de configuração mantém apresentação sem inventar destino. |
| PP-003-019 | To Deploy | S15 oferece o mesmo QR e link da resposta autorizada. |
| PP-003-020 | To Deploy | PNG com nome quebrado por grafemas e URL legível; filename scrapbook-evento-{code}.png. |
| PP-003-021 | To Deploy | Anchor removido; object URL revogada após 1s ou unmount; callbacks antigos descartados. |
| PP-003-022 | To Deploy | Portal no body, html/body brancos, nome/link/QR 70mm e controles excluídos no print. |
| PP-003-023 | To Deploy | Impressão na própria página, before/afterprint, cleanup e orientação de menu na indisponibilidade. |
| PP-003-024 | To Deploy | Geração/exportação/cópia independentes; trava durante exportação impede download duplicado. |
| PP-003-025 | To Deploy | Output/nomes visíveis/foco e feedback textual; nenhum sucesso falso de download/impressão. |
| PP-003-026 | To Deploy | PublicEventLinkPolicyTests e EventSharingServiceTests: origem, código, ausência e autorização. |
| PP-003-027 | To Deploy | Fixture HTTP válida de dois responsáveis; outro ator recebe 404. |
| PP-003-028 | To Deploy | Testes recusam código como PK e privado sem identidade; público continua anônimo. |
| PP-003-029 | To Deploy | Vitest: Clipboard resolvido/negado/indisponível; Playwright: seleção manual após negação. |
| PP-003-030 | To Deploy | Unitários e browser: falha de geração, PNG nulo/inválido, concorrência e descarte de recursos. |
| PP-003-031 | To Deploy | jsQR 1.4.0 + pngjs 7.0.0 decodificam bitmap, captura de prévia e PNG baixado. |
| PP-003-032 | To Deploy | URL decodificada abre outro contexto anônimo e apresenta o mesmo evento. |
| PP-003-033 | To Deploy | Print media contém só nome/link/QR, fundo branco; captura decodificada em dois browsers. |
| PP-003-034 | Test; incompleta | Aparelhos reais e impressão física não executados. Protocolo abaixo; sem evidência fabricada. |
| PP-003-035 | To Deploy | Origem HTTPS sintética/config ausente/Host adulterado e retorno ao local aprovados, sem rede externa. |
| PP-003-036 | To Deploy | FDD, READMEs de quatro componentes, ciclo, revisões, falhas/retestes e capturas documentados. |

## Pendência PP-003-034

Usar uma origem HTTPS realmente alcançável pelo celular; loopback no aparelho aponta para ele próprio. Registrar modelo/SO, câmera ou leitor/versão, navegador, origem/ambiente e condições de luz/distância. Ler QR da tela e do PNG; imprimir em escala 100%, conferir o QR de 70mm e ler o papel com os aparelhos disponíveis. Comparar o destino com o link textual e conferir evento correto em navegação anônima, sem acesso administrativo. Registrar falhas e o reteste com tamanho/condições corrigidos quando necessário. A inspeção digital aprovada não preenche esse registro.

## Falhas, revisões e operação

Quatro falhas intermediárias preservadas em [falhas e correções](../../testing/2026-11-13_15-57-13/falhas-e-correcoes.json): spy de clique restaurado pelo runner; fixture sem joined_at; fixture de segundo responsável sem convite aceito exigido pelo banco; proxy ainda convergindo após recriar a API. Correções ficaram no setup/testes, sem enfraquecer regra de banco ou acrescentar retry de escrita. As execuções finais passaram. Inspeção do print também corrigiu fundo externo bege para branco, com asserção e reteste.

[Revisões de código](../../testing/2026-11-13_15-57-13/revisoes.json) identificam service/UI/tests/infrastructure; documentação consolidada em commit posterior da mesma feature. Publicação futura: API/configuração antes da UI; recuperação: reverter UI, depois API/config, preservando banco. Sem novas migrations (47 changesets existentes), libs versionadas, Cognito, convite funcional ou AWS. Stack persistente parada, volumes preservados; laboratórios descartáveis removidos. Sem merge/deploy de PP-STORY-003.
