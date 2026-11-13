# Plano: PP-STORY-003

Plano consolidado das 36 subtarefas do [FDD](../../fdd/FDD-Compartilhamento-do-Evento/FDD-Compartilhamento-do-Evento.md), distinguindo teste planejado e executado.

## Objetivos

Conferir URL canônica única, autorização dos dois responsáveis, QR reutilizável público/privado, cópia/fallback, PNG e composição de impressão. Preservar acesso anônimo, isolamento de dados, leitura sem gravação e regressão da gestão/apresentação.

## Ambiente e critérios

Unitários xUnit/Vitest nos projetos de aplicação. HTTP, PostgreSQL real, QR e Playwright em scrapbook-tests, acionados manualmente em Compose com nomes, portas, credenciais e volumes exclusivos. Aplicar as mesmas 47 migrations; nunca usar o volume persistente ou acessar alvos externos. Origem HTTPS sintética só para comparar strings.

Gerar QR preto/branco, correção M, quatro módulos de margem e 1024px. Decodificar com jsQR, independente de qrcode: bitmap, captura, PNG e mídia print devem retornar exatamente a URL configurada. Conferir nome/link/QR na impressão sem controles administrativos e abrir destino em contexto anônimo. Chromium/WebKit emulados não representam aparelhos reais. PP-003-034 exige câmera/leitor e papel reais, permanecendo incompleta sem execução.

Regressão: acesso direto/reload/nova aba, apresentação responsiva e gestão/health. Perfil 4G customizado com 10/2Mbps decimais, latência mínima 80ms, CPU 4×, 390×844, cinco contextos novos/cache desligado; nome/mensagem/capa/QR prontos até 3s por amostra. Carga de leitura pública: dez clientes/60s, zero erros e p95 até 500ms.

Registrar comandos, revisões, medições, falhas/retestes, capturas revisadas e transições observadas. Não publicar logs brutos, segredos ou bibliotecas. Preservar pendência PP-002-029 de leitor de tela real. Sem merge da nova feature ou deploy.
