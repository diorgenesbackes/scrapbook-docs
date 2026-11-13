# Execução: PP-STORY-002

[Plano](1-plano.md) · [Relatório por tarefa](../../fdd/FDD-Apresentacao-Publica-do-Evento/relatorio.md)

## Base e revisões

[PP-STORY-001 integrada em cinco mains](integracao-PP-STORY-001.json) por solicitação do responsável, com merge commits e árvores iguais às versões já validadas. Feature/PP-STORY-002 criada dessa base; sem merge automático da nova US.

- scrapbook-service: `ae318ace6c3fd8b88ccc63368d3ce7ede09a6ce0`.
- scrapbook-ui: `f24722d0b904cbbc0cfee177f2950ce3207dbb4a`.
- scrapbook-tests: `a820723a8dd6ea3cda9d3325ee2abba738d54b88`.

Documentação/transições são consolidadas em commit posterior da mesma feature. infrastructure e liquibase não tiveram alteração nesta US.

## Comandos e resultados

- `dotnet build ScrapBook.sln --no-restore --disable-build-servers`: aprovado, zero avisos/erros. `dotnet test ScrapBook.sln --no-build --no-restore`: 104 testes aprovados, zero ignorados. Dependências/lockfiles preservados.
- `npm run check`: lint/build aprovados, 55 testes aprovados. Nenhuma falha funcional identificada nesta leva.
- `python3 scripts/stack_local.py build`: imagens construídas; a stack persistente não foi iniciada.
- `python3 public_presentation.py`: [resultado original](public-presentation-report.json). HTTP altera/restaura mensagem pela API de gestão e verifica resposta pública seguinte; valida projeção exata, privacidade, OpenAPI, autorização e snapshots. 11 testes de apresentação/performance aprovados em 8,3s e 14 de gestão/health em 8,5s. Sem skips/retry. [Resumo de navegador](browser-summary.json).
- Carga no mesmo laboratório: 1004 eventos, dez clientes, 60s; 269199 requests, zero erros, p95 3,367ms e máximo 31,343ms. Metas locais atendidas. Não mede AWS ou uploads.
- `python3 public_event.py --functional-only`: [regressão do acesso anterior](regressao-acesso-publico.json). Mesma API/UI; seis testes públicos (2,9s) e 14 de gestão/health (8,2s) aprovados. Carga explicitamente não repetida. Total de 31 cenários distintos; os 14 da gestão foram executados nos dois laboratórios.
- Parsing Python, whitespace, tokens idênticos ao guia e capa de 823 bytes conferidos. Capturas inspecionadas visualmente.

## Performance 4G e acessibilidade

[Perfil/medições](performance-4g.json): Chromium 158.0.8127.18, download 10Mbps, upload 2Mbps, latência mínima de headers 80ms, CPU slowdown 4×, 390×844, cache desabilitado/contexto novo por amostra. Página pronta inclui nome/mensagem visíveis e capa carregada. Tempos 752/754/751/739/747ms, máximo 754ms (meta 3000ms). Cinco amostras locais, sem inferência estatística sobre produção. WebKit 27.2 para a composição móvel emulada.

Teclado/foco/skip link, texto 200%, largura sem overflow e árvore semântica aprovados. WebKit usa Option+Tab para incluir links, conforme diagnóstico da entrega anterior. **PP-002-029 permanece em Test/incompleta: leitor de tela real não foi executado.** Não declarar conformidade WCAG completa ou teste em aparelho físico.

## Privacidade, dados e limpeza

Queries projetam somente code/name/welcome_message; serviço acrescenta somente mensagem efetiva e referência padrão. API não expõe IDs/permissões/recebimentos, nem URLs privadas. React renderiza markup como texto. Falha de imagem mantém apresentação. Mensagem completa e nome longo preservados. Snapshots das tabelas de aplicação idênticos após leitura/navegador/carga; fixtures e regressão privada fazem gravações intencionais separadas. Logs amostrados sem código ou texto de evento.

Containers/volumes exclusivos removidos no finally. Serviços persistentes e portas 5173/5080/55432 permanecem parados, volumes preservados. Sem migração nova, publicação AWS ou merge da nova história.

## Evidências

[Desktop](public-presentation-chromium-desktop.png) · [WebKit móvel](public-presentation-webkit-mobile.png) · [Revisões](revisoes.json) · [Transições UTC](transicoes.json).

Aceite humano, leitor de tela real e revisão visual integral permanecem separados dos testes automatizados locais.
