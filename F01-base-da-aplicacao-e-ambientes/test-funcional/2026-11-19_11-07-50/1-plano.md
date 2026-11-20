# Plano de validação local da F01-STORY-002

Os cenários usam um projeto Compose descartável com portas livres em loopback, credenciais temporárias e dados sintéticos. A base do Liquibase vem da F01-STORY-001. [História](https://trello.com/c/nv7cVIT2) · [Spec da F01](../../spec.md#7-estratégia-de-testes) · [Aceite no PRD](../../../PRD.md#9-critérios-de-aceitação)

| Critério | Ação | Resultado esperado |
| --- | --- | --- |
| CA1 · Execução local | Iniciar o serviço sobre um banco vazio, conferir o catálogo, preparar com Liquibase e acessar API e UI diretamente e pelo proxy. | A API mantém vitalidade no banco novo e não cria o schema. A preparação é explícita. As portas usam loopback e API e UI executam sem root. |
| CA2 · Recuperação | Consultar saúde antes da migração, parar PostgreSQL, restaurar a dependência e recriar os contêineres após inserir uma conta de teste. | `/health` responde `200` durante as falhas do banco. `/health/ready` responde `503` antes da migração e durante a indisponibilidade. Retorna a `200` após preparo e recuperação. Dados e chaves persistem. |
| CA3 · Configuração | Iniciar com conexão inválida e tentar habilitar a identidade local em `Production`. Verificar o comportamento padrão de autenticação nos unitários. | O processo recusa opções incompatíveis e identifica a opção sem revelar seus valores. A identidade local vem desabilitada. |
| CA4 · Interface | Abrir início e fallback, testar o link de atalho e a retentativa por teclado, verificar reflow, estados textuais, axe e a matriz manual. | A navegação e o foco seguem a ordem esperada. A falha de prontidão permite repetir. Os resultados distinguem motores automatizados de navegadores, dispositivos e leitores de tela verificados manualmente. |

Executar restore travado, build e unitários no serviço. Executar instalação travada, lint, build e unitários na UI. Executar a [suite local](../../../../scrapbook-tests/README.md) com `python3 docker_stack.py --browser` para Compose e Playwright. Na matriz manual, testar as duas versões principais mais recentes de Chrome, Safari, Edge e Firefox nas plataformas aplicáveis e conferir teclado, foco, contraste e anúncio de estados com leitores de tela.
