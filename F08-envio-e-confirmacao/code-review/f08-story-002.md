# Code review da F08-STORY-002

A revisão independente avaliou `2f8a871` no serviço, `c8ad301` na UI, `c8813f9` no Liquibase e `8dda194` na infraestrutura. Conferiu credencial de tentativa, assinatura do PUT, deadline de Brasília, propagação do `VersionId`, transação final, rollback, CORS e recuperação após interrupção. [Plano de aceite](../test-funcional/2026-11-25_14-54-00/1-plano.md)

| Achado | Resolução no candidato |
| --- | --- |
| Dependência HEIC | Atualização para Magick.NET 15.1.1 com lockfiles regenerados. O processo de decodificação tem saída limitada e diretório temporário privado. |
| Elegibilidade da limpeza de expirados | A limpeza de versões com prova usa o prazo de conclusão como âncora da tentativa expirada, além da idade da evidência individual. |
| Transferência sem avanço | O navegador interrompe após 30 s sem novos bytes e consulta a mesma tentativa. Indisponibilidade explícita da mídia mantém a tentativa admitida em pausa para consulta e retomada. |
| Origem de upload ausente | A composição permanece disponível. A UI habilita Enviar para foto quando a origem de upload está configurada. |
| Rolagem a 320 px após reiniciar o editor | A largura do seletor de arquivo foi limitada à área disponível. |

A prova automática nativa, a segurança efetiva do runtime do decoder e a recuperação operacional da DLQ exigem evidência em homologação antes da ativação. Para versões sem prova, o serviço conserva o resultado anterior e a evidência pendente. [Correção do decoder HEIC](https://github.com/dlemstra/Magick.NET/releases/tag/15.1.1) · [Base de mídia](../dev-infra/f08-story-002.md)

Na revisão do candidato, o serviço `20097b0` restaurou dependências com os lockfiles, compilou com zero avisos e passou em 179 unitários. A UI `c8005e7` passou em lint, build e 126 unitários. O npm registrou o aviso de script de instalação de `fsevents` durante a instalação no macOS. Os nove testes da infraestrutura e o `cfn-lint` passaram com a configuração KMS. O ensaio de navegador e a decodificação real da fixture passaram após as correções. [E2E](../test-e2e/f08-story-002.md)
