# Operação local — Gestão de Eventos

Estado: implementação validada na feature, sem promoção/deploy. [Story](https://trello.com/c/kIAhFOno) · [matriz de testes](3-testes.md).

## Preparação e inicialização

Manter os seis repositórios como diretórios irmãos, com `feature/GE-STORY-001` nos cinco componentes alterados e Liquibase main inalterado. Instalar .NET SDK 10.0.401, Node 26.13.0/npm 11.20.0, Python 3.13.15, Docker e Liquibase 4.33.0/Java.

Em scrapbook-service, executar restore locked, build e unitários. Em scrapbook-ui, `npm ci` e `npm run check`. Em scrapbook-infrastructure:

```bash
python3 scripts/liquibase_local.py init
docker compose up -d --wait --wait-timeout 60 postgres
python3 scripts/liquibase_local.py update
python3 scripts/events_local.py provision-runtime
python3 scripts/events_local.py bootstrap-actor --actor owner
python3 scripts/events_local.py run-api --enable-development-identity --no-build
```

Em outro terminal, a partir de scrapbook-ui, `npm run dev`; abrir `http://127.0.0.1:5173/events`. API: `http://127.0.0.1:5080`; OpenAPI: `/openapi/v1.json` em Development. A raiz do frontend mantém o botão de saúde.

O bootstrap idempotente devolve o ID real e o armazena privadamente; nunca assumir ID 1. Conexão DML e identidade entram por ambiente do servidor, sem segredos nos argumentos. Cliente não escolhe ator. Origem padrão: http://127.0.0.1:5173, configurável explicitamente em DevelopmentIdentity:AllowedOrigins; acesso local sem Origin é permitido. Flag ativa fora de Development impede startup.

## Conferência e recuperação

`/health` confirma somente que a API responde. Events sem identidade válida retorna 401. Para conferir prontidão funcional, consultar a listagem; não usar saúde como comprovação do banco.

Após timeout/resposta perdida de escrita, não repetir automaticamente. Conservar a página/rascunho e consultar listagem/detalhe; eventos podem compartilhar nome/data. Em conflito, recarregar/comparar e somente então decidir uma nova gravação. Os dois botões de salvar são operações independentes.

`Ctrl+C` encerra API/frontend. Reiniciar sem a flag desativa a identidade simulada, mantendo health. `revoke-runtime` desativa LOGIN sem apagar eventos; `provision-runtime` reaplica grants da credencial reservada. `rollback-actor --actor owner` só remove conta sem referências; não contornar FKs excluindo dados.

Para parar PostgreSQL, `docker compose down` preserva volume; `up -d --wait` restaura o serviço. Não usar down -v nem rollback da base populada. Retornar a código anterior/desabilitar acesso preserva schema/dados, mas versão anterior à story não oferece gestão de eventos.

Chaves de cursor em .local/cursor-keys devem permanecer persistentes e privadas. Removê-las invalida cursores anteriores. Credenciais, chaves e logs ficam fora do Git; arquivos privados 0600/diretórios 0700. Logs técnicos locais não comprovam retenção gerenciada de 180 dias; auditoria usa o default físico de retenção existente.

## Revisão e próxima promoção

Os cinco componentes têm commits de feature rastreados no relatório. Não há PRs/promoção dev/hlg/main, pipeline AWS ou release implantada nesta execução. Próxima etapa: revisão do responsável e definição da promoção conforme Git Flow; identidade simulada não pode ser publicada como autenticação real. Não anunciar autenticação, convites, capa customizada ou galeria como entregues.
