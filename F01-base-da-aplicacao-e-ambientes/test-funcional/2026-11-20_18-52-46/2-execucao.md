# F01-STORY-003 · Execução da validação funcional

A execução encontrou um bloqueador nas imagens candidatas da API e do Liquibase. O processo padrão de cada imagem precisa ler o bundle da CA do RDS para operar com `VerifyFull`. Os diretórios criados para o bundle ficaram com modo `0444`, sem permissão de travessia. A leitura falhou nas imagens sem montagens de teste, inclusive nas duas imagens Linux x86-64 previstas para ECS. [Dockerfile da API](../../../../scrapbook-service/Dockerfile#L20) · [Dockerfile do Liquibase](../../../../scrapbook-liquibase/Dockerfile#L13)

## Bloqueador · Leitura da CA do RDS

- **Esperado.** O usuário do processo lê o bundle fixado na imagem e valida o certificado apresentado pelo banco.
- **Observado.** A API usa o usuário `app` (UID 1654) e o migrador usa `liquibase` (UID 1001). `ls -ld` mostrou `dr--r--r--` nos diretórios `/app/certs` e `/opt/scrapbook/certs`. A leitura direta por esses usuários saiu com código 1 e `Permission denied`. O migrador também saiu com código 1 na pré-validação JDBC. Um diagnóstico independente com o mesmo driver confirmou que a abertura de `/opt/scrapbook/certs/rds-us-east-1.pem` foi recusada.
- **Impacto.** O migrador não aplica o changeset pelo entrypoint da imagem. A API não consegue usar o bundle obrigatório com verificação de hostname. Para reproduzir no build local, execute `head -c 0` sobre o bundle, usando o usuário padrão da imagem e sem montar arquivos adicionais.

## Matriz de resultados

| Critério | Verificação | Observado | Resultado |
| --- | --- | --- | --- |
| CA1 · Fundação local | `python3 -B docker_stack.py --browser` | Em 54,3 s, o runner observou vitalidade 200 e prontidão 503 antes do Liquibase. A instalação deixou a API pronta. A perda do PostgreSQL derrubou a prontidão e a recuperação preservou os dados e volumes da execução. O runner saiu com código 0. | Aprovado no laboratório local |
| CA1 e CA3 · Imagens TLS | Leitura do bundle como usuário padrão e execução do entrypoint do migrador | A leitura falhou nas duas imagens x86-64. O driver JDBC retornou `08006` com causa `Permission denied`. | Reprovado |
| CA1 e CA3 · Sequência com CA sintética | Diretório de certificados descartável montado com modo `0755` nas duas imagens | A imagem Liquibase x86-64 recusou modo TLS insuficiente e hostname incorreto. Com nome válido, aplicou 24 tabelas e registrou um changeset. A API ARM64 passou de 503 para 200 na prontidão, com sessão TLS observada no PostgreSQL. O CORS só respondeu com a origem autorizada. A reaplicação preservou a única entrada do histórico. O ensaio saiu com código 0. | Aprovado no laboratório com a montagem de teste |
| CA2 · Regressão de navegador | Playwright e Axe na stack local | Dez casos passaram em cinco projetos. O relatório JSON confirmou zero falhas, skips e intermitências. | Aprovado no laboratório local |
| CA2 e CA4 · Pacote de publicação | Scripts reais de build e verificação do ZIP | O build e o verificador passaram. O ZIP continha `release.json` com a revisão e versão sintéticas esperadas. Foram conferidos SHA-256, origem HTTPS da CSP, cabeçalhos e regra de fallback que preserva manifesto e assets. | Aprovado para o pacote local |
| CA3 e CA4 · Simulação da publicação | `verify_workflows.py` com os três checkouts | 27 cenários passaram, incluindo vínculo com a tentativa, bloqueio de ativação sem migração, validação do digest, verificação do ZIP e reconciliação de migração pendente. | Aprovado em simulação |
| CA4 · Reserva de versão | Testes em remoto Git bare descartável | Cinco testes passaram. Foram conferidas a disputa entre `hlg` e `main`, a identidade da tentativa e a recuperação da reserva. | Aprovado no remoto Git descartável |
| CA1 · AWS | VPC, RDS, isolamento de armazenamento e identidades, prontidão do ALB | O candidato ainda não está em `hlg` e o ambiente AWS de homologação não foi identificado na configuração local. | Bloqueado |
| CA2 · AWS | HTTPS, Amplify, ALB e calibração do WAF | Os serviços da revisão atual ainda precisam estar publicados em um alvo identificado. | Bloqueado |
| CA3 · GitHub Actions e AWS | Execução real com OIDC, implantação e ordem entre banco e tráfego | As revisões remotas `hlg` da API (`a498ab3`), UI (`f10da17`) e Liquibase (`71819af`) ainda não contêm o workflow de publicação da história. | Bloqueado |
| CA4 · GitHub Actions e AWS | Tags protegidas, execução hospedada e evidência da publicação | A evidência desta rodada cobre Git local e respostas AWS simuladas. A comprovação do fluxo remoto depende do candidato publicado. | Bloqueado |

A primeira tentativa do laboratório TLS não recebeu uma porta publicada na rede Docker interna. O reteste usou um bridge descartável com a API em loopback e o PostgreSQL sem porta publicada no host. As duas imagens sem a montagem de teste foram verificadas separadamente. Os contêineres e a rede descartáveis foram removidos. A instalação local do npm avisou sobre `allowScripts` para o opcional `fsevents@2.3.3`. Os testes de reserva emitiram avisos esperados de clone inicial vazio.

[Plano deste ciclo](1-plano.md) · [Resultado Playwright](../../test-e2e/2026-11-20_18-52-46/execucao.md) · [História no Trello](https://trello.com/c/Ji6EyWMB)
