# Revisão técnica da F10-STORY-002

A revisão independente examinou os commits de serviço a partir de `e9b6fe4` e a infraestrutura no candidato `ebb6674`. Reavaliou a produção em `08283a1` e os testes de associação em `346b15e`. A última alteração do serviço, `5e8c5d3`, só ajustou a formatação de três arquivos de teste. [Cartão da story](https://trello.com/c/I4MI0oAu)

## Achados verificados e correções

- O decodificador inicial aceitava uma amostra AAC-LC válida com tag de localização ISO 6709. O adaptador agora examina tags do contêiner e das faixas antes de publicar a variante. O reteste independente aceitou a amostra limpa e rejeitou a amostra com localização. Os unitários exercitam também a tag QuickTime.
- O limite anterior de 512 MiB no Compose ficava abaixo dos 778.000.000 bytes da execução simultânea de vídeo F08 e áudio F10. O Compose passou a limitar 1 GiB por réplica. A documentação registra a reserva necessária na tarefa publicada. [Infraestrutura](../dev-infra/f10-story-002.md)
- O resultado retornado pelo provedor passa pela conferência com a tentativa e o plano persistidos antes do vínculo por callback ou consulta. Divergências de role, token e destinos, quando presentes, mantêm a reconciliação. ID, ARN, fila e chave de correlação sustentam o vínculo quando campos opcionais são omitidos. A promoção exige a versão esperada do objeto e uma faixa privada válida.
- O redrive faz parte da etapa de recuperação. A política atual de entrega pela regra EventBridge precisará de autorização específica para redrive e ensaio em AWS nessa etapa. [Plano da F10](../plan.md#fase-4--entregar-recuperação-estado-e-falhas)

O candidato recebeu parecer técnico favorável à integração em `dev`. O contrato e as permissões efetivas do MediaConvert seguem como critérios de homologação.

## Checks locais

| Check | Resultado |
| --- | --- |
| Restore .NET em modo locked, build e xUnit | Build com zero warnings. 577 unitários aprovados, sem falhas ou casos ignorados. São 130 casos novos. |
| Formatação | Verificação de espaços nos C# alterados aprovada após ajuste nos testes. A verificação geral registra 24 diagnósticos preexistentes de espaços e sugestões informativas. |
| UI `57f4e63`, lint, build e Vitest | Lint e build aprovados. Os 188 testes passaram. |
| Infraestrutura | 21 testes Python aprovados, quatro novos sobre as políticas e a fila. |
| `git diff --check` | Aprovado nos quatro repositórios afetados. |

A coleta dos unitários registrou 331 de 368 linhas e 272 de 316 ramificações exercitadas no serviço de preparação, 123 de 127 linhas no adaptador S3 e 188 de 200 no adaptador MediaConvert. A validação integrada complementa os testes de processo, PostgreSQL, S3 e mensageria.
