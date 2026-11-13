# Plano: PP-STORY-005

[FDD](../../fdd/FDD-Recuperacao-da-Pagina-Publica/FDD-Recuperacao-da-Pagina-Publica.md). Plano consolidado das 33 subtarefas: erros de links, rede, timeout, indisponibilidade, correlação segura e recuperação de consultas sem misturar eventos.

## Base e preparação

Parar stack com volumes preservados; auditar branches locais/remotas dos seis repositórios, integrar PP-STORY-003/004 e enviar as mains. Conferir árvore igual à PP-STORY-004 validada. Executar pull de main após os pushes e antes de criar/codificar feature/PP-STORY-005. Preservar QA anterior PP-003-034 e PP-002-029, sem declarar deploy.

## Verificações

Unitários xUnit: Problem Details, correlação, sanitização/logs nominais e abandono do cliente. Vitest: classificações, timeout 6s, sinal composto, contrato inválido, Retry-After, cooldown, foco e duplicação, cancelamento e limite de revalidações vencidas. Sem retry de escrita ou dependência nova.

HTTP real em Compose aleatório: vazio/malformado/longo/encoding/ausente; 404/500/503, Content-Type/no-store/correlação; revogar/restaurar SELECT, NOLOGIN/LOGIN e lock exclusivo somente do runtime/banco descartáveis. Provider 4s/API 5s, recuperação posterior e snapshots sem escrita. Rejeições do edge antes do pipeline não têm garantia de JSON da API. Nginx local precisa mostrar uma página estática guest segura em 400 de documento inválido, sem refletir URI ou redirecionar.

Chromium/WebKit emulados: rede/HTML 503/429, espera sem auto retry, timeout e resposta tardia, ação manual e foco, recuperação real sem reload com janela atual, capa/QR e contexto privado de compartilhamento. WebKit usa Option+Tab conforme convenção do projeto. Árvore semântica/teclado não comprovam leitor de tela real.

## Regressão e critérios

Acionar manualmente janela, QR, apresentação/gestão e acesso direto/reload/nova aba. Página com capa/QR/disponibilidade pronta até 3s no perfil 4G existente: 10/2Mbps decimais, 80ms de latência mínima, CPU 4×, 390×844, cinco contextos novos/cache desligado. Carga de leitura pública: 1004 eventos, dez clientes/60s, zero erros e p95 até 500ms. Não medir upload ou AWS por inferência.

Registrar revisões, matrizes, falhas/retestes, capturas e transições nativas UTC por PP-005-NNN. Não versionar logs brutos, manifests privados, senhas ou libs. Cada laboratório limpa somente seus próprios recursos; não iniciar a stack persistente ou integrar a nova feature em main automaticamente.
