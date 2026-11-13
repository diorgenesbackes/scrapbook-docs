# Plano: PP-STORY-002

Plano consolidado das 33 subtarefas e do contrato definido durante a implementação. [FDD](../../fdd/FDD-Apresentacao-Publica-do-Evento/FDD-Apresentacao-Publica-do-Evento.md).

## Objetivos

Validar apresentação pública, regra compartilhada de boas-vindas, capa permitida/fallback, Unicode seguro, tokens, layout responsivo e revalidação. Preservar código, isolamento privado, leitura sem gravações e compatibilidade do acesso público anterior.

## Ambiente e critérios

Compose real com recursos/credenciais/volumes exclusivos; mesmas 47 migrations, runtime DML e 1000 eventos de carga mais quatro fixtures. Unitários nos projetos de aplicação; testes manuais em scrapbook-tests. Nunca usar o volume persistente ou alvos HTTP externos.

Chromium/WebKit emulados em 320/390/768/1440px, texto 200%, teclado/foco e árvore acessível. Esta última não substitui teste com leitor real de PP-002-029. Perfil 4G customizado: 10/2Mbps decimais, latência mínima 80ms, CPU 4×, 390×844, cinco contextos novos/cache desligado; página pronta até 3s em cada amostra. API: dez clientes/60s, zero erros, p95 até 500ms.

Registrar resultados agregados/capturas revisadas, hashes, pendências humanas, comandos e tempos reais. Não versionar manifestos com links, segredos, bibliotecas ou logs brutos. Sem merge da nova feature ou deploy.
