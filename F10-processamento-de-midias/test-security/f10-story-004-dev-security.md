# Validação defensiva da F10-STORY-004 em dev

A validação de 28/11/2026 cobriu a recuperação operacional de mídias, a preservação das versões prontas e do original, o isolamento por evento, os limites da identidade operacional no PostgreSQL e o recibo público da F08. [Especificação da F10](../spec.md) · [Comando operacional](../dev-back/f10-story-004.md) · [Cartão](https://trello.com/c/dLVALXU9)

| Componente integrado | SHA |
| --- | --- |
| Serviço | `3834529e86e1123a4c3119c67be5395478ea6612` |
| Liquibase | `ac0e8517af47cdf4a61b81c715b0e8544040ecde` |
| Infraestrutura | `8f6dd8cfb4226c467c48fc63e34667a82fb4b17a` |
| Testes | `4a720663ae2f1a398ba941bb2f7c10a6cc674822` |
| UI | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |
| Documentação de base | `c6eba10e48ece6d0b54b2635539ae3eef5591223` |

A análise de fonte usou os commits acima. Os cenários dirigidos usaram Docker Compose descartável, PostgreSQL real, Moto para S3/SNS/SQS, fotos sintéticas e eventos de teste. A fixture da suíte injeta um relógio controlado no build descartável para avançar as janelas de recuperação e retenção, com identidade local habilitada explicitamente para a criação dos eventos. O CLI opera com um login PostgreSQL dedicado à fixture. O ensaio HTTP partiu do contrato e do conhecimento prévio do código.

## Achado e reteste

**F10-SEC-01, escopo IAM excessivo para exclusão por versão.** No candidato `e693c17db4568dde0c05fdcc0a75170176da1141`, a role de limpeza podia solicitar exclusão em todos os prefixos de derivados de foto e transcodificação. O mesmo namespace contém versões já promovidas. O runner filtrava as referências prontas. A garantia ficava restrita ao caminho da aplicação. O achado foi confirmado estaticamente, com gravidade alta para a ativação da limpeza.

O primeiro ciclo corrigiu a política no commit `7acbf9a43079055c2feff82a6021a835235b8b63` e a seleção do runner em `e2e076cd592388830d2261f0a2443de4ea174ed6`, ambos no integrado. O template admite uma única chave e um único VersionId aprovados e nega explicitamente exclusão sem versão, fora da chave ou fora da versão. O CLI exige objeto selecionado na configuração AWS. O serviço confere o resultado exato antes de reivindicar o trabalho. O responsável pelo provisionamento verifica a elegibilidade do par, fora do escopo da role que executa a limpeza. Os 23 testes de infraestrutura passaram e `git diff --check` passou nos commits de correção. F10-SEC-01 encerrado na revisão estática. [Referência de autorização do S3](https://docs.aws.amazon.com/service-authorization/latest/reference/list_s3.html)

## Evidência no integrado

| Perspectiva | Controle, identidade e evidência | Resultado |
| --- | --- | --- |
| White box | O CLI exige habilitação explícita e conexão própria, e o repositório identifica o operador pela sessão PostgreSQL. A abertura de ciclo, o redrive e a seleção de limpeza cruzam evento, arquivo e estado comprovado. A projeção de variantes prontas usa evento, registro, arquivo, perfil ativo e tentativa concluída. | Aprovado no código integrado. |
| White box | As políticas SQS delimitam a fila de retorno, o principal operacional e a entrega pelo serviço. O IAM de limpeza exige chave e versão específicas. O armazenamento de mídia é privado e versionado pelo template. | Aprovado nas verificações estáticas, 23 de 23 testes de infraestrutura. |
| Grey box | `test_cross_event_and_audit` confirmou negação de redrive e reabertura em outro evento, manutenção do mesmo trabalho e fato canônico, deduplicação restaurada após 31 dias com efeito comprovado e recusa quando o efeito faltava. A simulação preservou o trabalho. A reabertura auditada avançou um ciclo e reentregas preservaram o resultado pronto. | Aprovado, código 0. |
| Grey box | `test_operator_gate_and_public_receipt` consultou os privilégios na sessão do operador. A associação herdada e o INSERT de auditoria estavam disponíveis. SET ROLE, ownership, CREATE, leitura de nome/legenda/hash de autorização e alterações de chave/versão estavam bloqueados, assim como SELECT, UPDATE, DELETE e TRUNCATE na auditoria. O CLI recusou aplicação com recuperação desabilitada e com login sem role. A auditoria continuou vazia. | Aprovado, código 0. |
| Black box guiado | Recibo sintético permitido com vínculo apresentou `state` e `confirmedAt`, com `Cache-Control: no-store`. O mesmo pedido sem credencial ou sob outro evento retornou 404 com `RESOURCE_NOT_FOUND`, incluiu `no-store` e omitiu estado, data, bucket, chave, versão e variantes. | Aprovado, código 0. |
| Grey box | `test_cleanup_and_public_receipt` verificou o prazo de 24 horas, o isolamento por evento e a simulação sem efeitos. A execução excluiu a versão persistida do artefato elegível e conservou as demais versões, a chave vizinha e as versões ambíguas. A intenção de limpeza concluiu com o VersionId esperado e sem delete marker. A auditoria registrou a reivindicação e a conclusão. | Aprovado, código 0. |
| Black box guiado | A leitura autenticada no S3 emulado confirmou a integridade dos checksums do original e das variantes prontas após a limpeza. A leitura anônima das mesmas versões foi recusada. O recibo público permaneceu íntegro e sujeito ao vínculo. | Aprovado, código 0. |

## Execução e limites

Os casos em `scrapbook-tests/media_recovery_stack.py` foram executados por um wrapper temporário para incluir as observações HTTP e as consultas ao catálogo. O caso de autorização e recibo levou 28,7 s, o de limpeza 38,6 s e o de redrive/reabertura 39,1 s. Cada processo retornou código 0. Ambiente Python 3.13.15, Docker CLI 29.8.2 com contexto Colima descartável, PostgreSQL 18.6 e infraestrutura emulada. O servidor Docker da fixture usava a versão 29.5.2. A suíte estática usou `env PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s tests -v` na infraestrutura.

As primeiras tentativas foram interrompidas na abertura de portas do sandbox ou na inicialização do Docker. A execução convergiu com o contexto explícito do Colima e os temporários sob `/private/tmp`. A asserção preliminar do cabeçalho tratava `no-store` como valor único. O reteste consultou as diretivas e aceitou o valor observado `no-cache,no-store`. Esses ajustes foram do laboratório e do wrapper.

O login PostgreSQL operacional é de confiança e seu uso SQL direto tem alcance mais amplo que o runner. O grant de estado em `storage_objects` alcança o estado de objetos por coluna. O grant de Outbox habilita leitura na tabela. A autorização por evento e a elegibilidade da limpeza são aplicadas pelo runner. O provisionamento mantém a conexão do operador separada e com acesso restrito à operação.

A homologação com dados sintéticos deve confirmar a recusa efetiva de exclusão fora do par autorizado, a preservação de originais e variantes prontas com IAM, o redrive nativo da DLQ e o comportamento do MediaConvert. Os testes locais avaliaram a configuração e a aplicação no laboratório.
