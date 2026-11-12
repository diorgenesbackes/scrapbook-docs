# Execução — preparação de Gestão de Eventos

Data: 2026-11-12. Início da fase Test: 20:04:48 de Brasília (`23:04:48Z`). Executor: Codex. Alvo: Compose PostgreSQL local e API/frontend em loopback; volume existente preservado.

## Revisões

- Service: `b86e996` (GE-001), `64c5739` (GE-005/006).
- Infrastructure: `061c408` (GE-002/003/004).
- Tests: `245d50d` (GE-007; pins de ferramentas acrescentados na consolidação).
- UI: `ea62ae4` (GE-008).

## Resultado

| Verificação / comando | Resultado |
| --- | --- |
| `dotnet restore ScrapBook.sln --force-evaluate` e build `--no-restore` | Pacotes fixados; build sem warnings/erros. |
| `dotnet test ScrapBook.sln --no-build --no-restore` | 5 aprovados, 0 falhas, 0 ignorados. |
| `npm run check` no frontend | Lint/build aprovados; 18 unitários em 3 arquivos aprovados. |
| `python3 run.py foundation` | Runtime sem superuser/CREATEDB/CREATEROLE/replication/BYPASSRLS/DDL/TEMP e sem escrita em Identity. DDL/TRUNCATE negados. |
| Transação manual como runtime | Inseriu evento + membro slot 1 + auditoria, validou constraints diferíveis e fez rollback integral da massa. |
| Repetição/reversão | ID de owner preservado na repetição; colisão com conta de outro namespace recusada; fixture sem referências removida com repetição segura. Revoke/provision do runtime preservou dados. |
| Inventário | 33 tabelas e 47 migrations preservadas. |
| `python3 run.py keys` | Cursor protegido lido após recriar provider com o mesmo diretório; permissão privada 0700 confirmada. Não é ainda o teste do cursor HTTP. |
| HTTP `/health` | `200`, JSON `{"status":"up"}`, `Cache-Control: no-store, no-cache`. |
| Playwright `browser/health.spec.ts` | 2 aprovados: Chromium desktop e WebKit móvel emulado, incluindo ausência de overflow horizontal. |
| Runner com destino remoto | Rejeitado no parsing antes de executar ferramentas ou fazer requisição. |
| Git | Dependências, binários, `.env`, caches e chaves locais excluídos. |

## Baseline operacional

`python3 run.py health-load --duration 5`: 10 requisições concorrentes, 39.156 respostas em 5s, zero erros, p95 **2,073 ms**, após aquecimento de 10 chamadas. Essa medição serve somente à regressão de /health, sem banco. Não comprova a meta de Events, 200 uploads, disponibilidade ou capacidade de produção.

## Ambiente e limites

.NET SDK 10.0.401, Node 26.13.0, npm 11.20.0, Docker 29.8.2, PostgreSQL 18.6. Playwright 1.65.0: Chromium 158.0.8127.18 e WebKit 27.2 em macOS arm64. WebKit móvel é emulação; não foi usado iPhone físico.

O sandbox bloqueou named pipes do runner .NET; a execução autorizada fora dessa restrição aprovou. Um warning de escape Python foi corrigido e a suíte foundation repetida com sucesso. O Playwright emitiu aviso de variáveis de cor do terminal, sem falha de teste.

Não executados nesta leva: CRUD e ACL HTTP de Events, políticas de abertura, concorrência de edição, cursor após restart real da API, carga H-03 de listagem/detalhe/atualização, navegadores em aparelhos físicos e implantação. Esses itens não são considerados aprovados por esta preparação.
