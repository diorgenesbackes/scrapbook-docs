# Modelagem do banco de dados

[Índice](README.md) · [Dicionário físico](DB-Modelo-Fisico.md) · [Diagrama ER](DB-Diagrama.md)

Data inicial: **2026-11-11**. Escopo original: modelagem conceitual, lógica e física da primeira versão do Scrap Book, sem scripts naquela entrega. **Adendo de 2026-11-12:** a [base Liquibase](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) materializa o modelo em scripts/rollbacks; não implementa integração EF nem aplica alterações em RDS.

## Fontes e status

Base: [PRD](../PRD.md), [HLD](../HLD.md), [SQL](../guidelines/sql.md), [Liquibase](../guidelines/liquibase.md), [Backend](../guidelines/backend.md), [APIs](../guidelines/api.md) e [Mensageria](../guidelines/mensageria.md). As regras de produto e arquitetura já aprovadas são preservadas. Os nomes físicos, estados auxiliares e decisões P-01 a P-07 abaixo são propostas de detalhamento, sem alegação de implementação ou aprovação adicional.

Aplicam-se as diferenças já registradas na [matriz PRD/HLD](../guidelines/README.md#compatibilização-entre-prd-e-hld): confirmação de e-mail no Cognito; recuperação por código de uma hora; travamento estrutural eventual com meta de 5 segundos; URLs já emitidas válidas por até 2 minutos; agendador somente para notificações. Não reintroduzir recuperação local por link de 30 minutos, travamento distribuído imediato ou abertura dependente de e-mail.

## Limites e fontes de verdade

| Schema / módulo | Dados próprios | Fronteira |
| --- | --- | --- |
| `identity` / Identity | Contas locais, sessões BFF, desafios OIDC e reconciliação de operações de conta. | Cognito continua dono de credenciais, confirmação de e-mail e recuperação. |
| `events` / Events | Eventos, responsáveis, convites, capas e obrigações de aviso de abertura. | Configuração de acesso e travamento pertencem a Events. |
| `records` / Records | Avisos de concordância, envios, arquivos, armazenamento, variantes e jobs. | S3 guarda os bytes; PostgreSQL guarda metadados, confirmação e coordenação. |
| `notifications` / Notifications | Intenções de e-mail e tentativas SES. | Aceite SES não significa entrega e não libera conteúdo. |

Cada módulo tem seu DbContext e suas próprias Outbox, inbox (`processed_messages`) e auditoria. API e consumidores continuam na mesma aplicação/serviço ECS. Não há schema de worker genérico, Redis, transação distribuída ou transação que grave dois módulos. Contratos internos fornecem leituras autorizadas; mensagens propagam fatos eventualmente.

**Proposta P-06:** FKs físicas entre módulos são limitadas a pais estáveis (conta, evento, convite e obrigação de aviso), com RESTRICT. Elas protegem referências no mesmo PostgreSQL sem autorizar leitura/escrita direta do outro contexto. Referências históricas, polimórficas e mensagens usam IDs lógicos. Exclusão extraordinária exige procedimento coordenado futuro; não usar CASCADE nem promessa de remoção já implementada.

## Modelo conceitual e cardinalidades

| Relação | Cardinalidade / regra |
| --- | --- |
| Conta → sessão / operação de conta | Uma conta possui zero ou várias; sessão pertence a exatamente uma conta. |
| Conta ↔ evento | Muitos para muitos por `event_members`; cada evento possui um ou dois responsáveis de igual permissão. |
| Evento → convite | Vários históricos; no máximo um pendente. Aceite liga uma conta ao slot 2. |
| Evento → capa | Vários uploads/históricos; zero ou uma capa atual. NULL seleciona o padrão da aplicação. |
| Evento → obrigação de abertura | Uma por membro e versão da política. Não é uma nova permissão. |
| Evento → registro | Zero ou vários; cada registro pertence a um evento. |
| Aviso → registro | Uma versão de aviso pode ser aceita em vários envios; cada registro referencia uma versão imutável. |
| Registro ↔ operação idempotente ↔ sessão | Exatamente uma de cada na criação/conclusão local; sessão também existe para texto, sem objetos S3. |
| Registro → arquivo | Texto: zero; foto de depoimento/vídeo/áudio: um; fotos de momento: de um a dez. |
| Arquivo → objeto / variante | Vários objetos técnicos; exatamente um original durável na confirmação; variantes por tipo e versão do perfil. |
| Arquivo → job → tentativa | Jobs por operação/perfil; várias tentativas históricas, no máximo uma ativa por job. |
| Objeto → limpeza | No máximo um job por ação; só objetos elegíveis, nunca originais confirmados. |
| Notificação → tentativa | Várias tentativas históricas; no máximo uma ativa. Uma obrigação produz uma intenção lógica por chave. |

Não existem tabelas de convidados cadastrados, lista oficial, senha, código de recuperação, QR code, URL assinada, rascunho offline, contagem pública, download, busca por remetente ou exclusão pela interface. QR e link são derivados do código público do evento/segredo do convite. A ausência de identidade de convidado é intencional: nomes iguais não significam a mesma pessoa.

## Decisões de detalhamento propostas

Estes valores detalham o desenho sem substituir requisitos já aprovados. Foram usados como parâmetros da base SQL solicitada em 2026-11-12; os contratos de aplicação devem respeitá-los, e alterações posteriores ao histórico aplicado exigem novas migrations.

| Referência | Proposta / motivo |
| --- | --- |
| **P-01 — textos** | Nome de conta, evento e convidado: 120 caracteres; boas-vindas: 2.000; e-mail: 320; versão de aviso: 32; códigos de erro: 80; MIME: 128; codec: 64; container: 32. O PRD não fixou esses limites. Texto de depoimento (5.000) e legenda (500) são requisitos existentes. Unicode deve ter a mesma contagem na UI/API/banco; sem truncamento silencioso. |
| **P-02 — unidades e capa** | MB decimal: foto 20.000.000 bytes, vídeo 500.000.000, áudio 50.000.000. Capa até 20.000.000 bytes é proposta específica, não limite aprovado no PRD. Duração em segundos `numeric(12,6)`; validar antes da coerção/arredondamento. Não introduz duração mínima de produto; arquivo vazio/inválido é rejeitado. Bitrates, dimensões finais para mídia vertical e consumo máximo do decoder permanecem parâmetros do perfil a validar. |
| **P-03 — identificadores de acesso** | Código público do evento: 32 bytes criptograficamente aleatórios em base64url de 43 caracteres, separado da PK. Convite/sessão usam segredos protegidos. O navegador gera antes do primeiro POST uma chave UUID de idempotência e um segredo aleatório separado; somente HMAC do segredo é persistido. Isso permite recuperar uma resposta perdida sem transformar UUID/ID sequencial em credencial. |
| **P-04 — controles técnicos** | Desafio OIDC: 10 minutos; autorização de replay/status de envio: 30 dias; identificador interno de chave criptográfica: 64 caracteres; payload de mensagem: objeto até 16 KiB, só referências. Esses são parâmetros propostos. Replay após o prazo de upload nunca permite enviar novos bytes. Leases, backoff, limites de tentativas e retenção SQS/DLQ ainda dependem de perfil de carga e contratos AWS. |
| **P-05 — limpeza** | A regra existente é iniciar limpeza de incompletos/rejeitados após 24h. Proposta: contar a partir do prazo final para incompletos, ou rejeição irreversível para rejeitados; reconciliar S3 antes de excluir. Nome/texto/legenda de registro nunca confirmado são saneados após essa elegibilidade. Capas rejeitadas/superadas seguem 24h após saída de uso. Segredos de sessões de acesso encerradas são apagados ao revogar/expirar; tombstones de sessões de acesso e desafios OIDC podem ser expurgados em 30 dias. Sessão de upload e operação de envio mantêm os metadados mínimos junto do registro, sem segredo após o prazo de replay. Evidência técnica/auditoria: 180 dias; chaves mínimas de deduplicação de fatos confirmados duram enquanto o fato existir. |
| **P-06 — integridade entre módulos** | FKs somente para pais estáveis, RESTRICT, contratos internos e commits locais. Referência ao registro que causou o travamento permanece lógica, para não desfazer o fato por limpeza extraordinária. |
| **P-07 — tempo e estabilidade** | Janela de upload é capturada no início autorizado; alteração posterior de evento não invalida um envio admitido. O congelamento da política de abertura é monotônico: antes de editar uma data de evento sem registros, avaliar o encerramento da janela anterior e persistir a trava efetiva. Isso impede reabrir a possibilidade de mudança livre movendo a data do evento. |

## Tipos, nomes e identidade

Todas as tabelas possuem `id bigint GENERATED ALWAYS AS IDENTITY`, PK simples nomeada. No .NET: `long`; em JSON: string decimal opaca. UUID fica em mensagens, correlação e chaves técnicas. Não conceder acesso pela imprevisibilidade ou existência de um ID.

Campos comuns são nomes em inglês, minúsculos, snake_case, sem prefixo redundante. Colunas de FK seguem `<origem>_<destino>_id`, acrescentando papel quando necessário; por exemplo, `invitations_issuer_event_members_id`. Constraints e índices são nomeados; o [dicionário](DB-Modelo-Fisico.md) especifica todos os campos, tipos, nulabilidade e relações.

`created_at`, `updated_at`, prazo de sessão, lease e instante de provedor são `timestamptz` (UTC). Data de evento, abertura, início/recebimento de envio e prazo de negócio são `timestamp without time zone` de Brasília. Não usar o timezone da máquina/conexão implicitamente. Ao comparar S3 UTC com 13h de Brasília, converter explicitamente usando `America/Sao_Paulo`; não aplicar offset fixo como regra universal.

Estados são `varchar` limitado e CHECK com catálogo, sem enum PostgreSQL nesta proposta; facilita evolução compatível. Relações e campos essenciais são tipados; `jsonb` é restrito a contrato de mensagens, snapshot de perfil técnico e auditoria com allowlist.

## Regras de negócio e persistência

### Contas e acesso

- Uma conta local representa um `sub` Cognito confirmado. `sub` é case-sensitive e tem escopo no emissor; adota-se um User Pool por ambiente, configurado fora da linha. Trocar o emissor exige migração/reconciliação, não reaproveitamento cego do valor. [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#IDToken).
- E-mail pendente não substitui o confirmado. `normalized_email` segue a configuração comprovada do User Pool; não assumir que converter todo endereço para minúsculas é sempre correto.
- Sessão local tem prazo absoluto máximo de sete dias. Cookie bruto não é persistido; tokens necessários ao BFF usam envelope criptográfico. Cada pedido verifica conta ativa, expiração, revogação e igualdade de `account_security_version` com `accounts.security_version`.
- Troca/recuperação de senha marca `revocation_pending` e invalida sessões localmente antes de concluir sucesso ao usuário. Persistir intenção e reconciliar Cognito fora da transação; falha incerta não reativa sessão antiga. Recuperação iniciada diretamente no Cognito precisa de sinal confiável para invalidação local; integração ainda deve ser comprovada.
- Não guardar senha, nova senha, código de confirmação/recuperação ou authorization code. `oidc_requests` guarda state/nonce protegidos, HMAC do vínculo ao navegador iniciador e verifier criptografado, com uso único; callback valida também o cookie temporário desse vínculo. Se o código externo já consumido não puder ser recuperado, reiniciar autenticação com segurança.

### Evento, responsáveis e convites

- Criar evento e membro slot 1 na mesma transação. UNIQUE `(evento, slot)` mais slot em 1..2 impede terceiro responsável, inclusive sob concorrência; UNIQUE `(evento, conta)` impede duplicidade. O slot não cria hierarquia de permissões.
- Dados públicos: nome, capa e boas-vindas. Padrões ficam em assets/configuração versionada da aplicação. Não há listagem, nomes de remetentes ou contagens públicas.
- Gerar convite com evento bloqueado por linha: revalidar vínculo e travamento; invalidar pendente anterior, depois criar novo com 24h de validade e request_key único. O índice parcial de pendência não usa relógio; expiração é materializada antes de substituir, mas sempre verificada no acesso.
- Aceitar convite bloqueia evento e convite na mesma ordem, revalida validade/estado, capacidade, conta confirmada e travamento; grava slot 2, uso do convite e auditoria juntos. Repetição pela mesma conta retorna o resultado seguro; outra conta não reutiliza o token.
- O primeiro registro **confirmado**, inclusive texto, grava fato e Outbox em Records. Events consome idempotentemente e trava dados/convites em transação própria. Não há teste cruzado atômico no banco para eliminar a janela eventual aceita no HLD. Mensagens de outros registros não reabrem o evento nem substituem a motivação de travamento já aplicada.
- Travamento estrutural cobre nome, capa, boas-vindas, data e hora; não elimina as mudanças de acesso permitidas. Uma capa ready selecionada é imutável, e a limpeza não a remove.

### Janela de envio e abertura

Para início do evento `D H`, abertura de envio = início menos 2h; fim de novos envios = `D+1 12:00`; fim de conclusão = `D+1 13:00`, sempre Brasília. Proposta de bordas: novo envio em **[abertura, 12h)**; conclusão admitida com evidência **até 13h inclusive**. Revalidar prazo depois de esperas relevantes.

O início é o recebimento pelo backend do pedido feito ao clicar Enviar após revisão. Seleção, gravação, relógio do navegador e URLs ainda válidas não estendem o prazo. A sessão guarda a janela autorizada. Recebimento físico integral em tempo pode ser validado/confirmado depois das 13h; atraso de callback ou processamento não prova atraso de upload.

Abertura futura é o padrão, sugerida para um ano depois do evento, com 29/02 → 28/02. A data escolhida é posterior ao dia corrente e ao encerramento dos envios, com liberação à meia-noite. Datas relativas ao “hoje” são validadas na operação; não criar CHECK que passe a invalidar uma linha apenas pela passagem do tempo.

| Estado de acesso persistido | Comportamento |
| --- | --- |
| `flexible` | Troca entre immediate/scheduled enquanto a janela livre não encerrou. O relógio pode tornar a trava efetiva antes de qualquer atualização da linha. |
| `scheduled_locked` | Janela terminou em scheduled, ou foi consumida a única transição posterior: modo/data ficam imutáveis. |
| `immediate_once` | Janela terminou em immediate: pode permanecer assim ou mudar uma única vez para scheduled_locked com data válida. |

`access_frozen_at` nunca é apagado por mudança estrutural de data. `access_version` aumenta quando a configuração de abertura muda, não por toda alteração de linha; invalida obrigações/notificações antigas. A tolerância até 13h não estende a liberdade de trocar o modo.

A autorização lê vínculo e configuração atual em cada acesso; em scheduled, só libera quando relógio >= opens_at. O agendador pode estar atrasado e o conteúdo já estar liberado. Após immediate → scheduled, impedir novas respostas de galeria/URLs; URLs emitidas continuam com validade de até dois minutos. O banco não recolhe conteúdo já entregue.

### Contribuições e confirmação integral

Cada envio congela nome/apelido, versão do aviso e instante de concordância. O navegador pode reutilizar nome/concordância por evento, mas o servidor recebe e valida a concordância a cada envio. Alterar nome exige nova concordância; não deduplicar pessoas por nome.

| Categoria | Formato | Quantidade | Limite por arquivo |
| --- | --- | --- | --- |
| Depoimento | Texto | Zero arquivos | 5.000 caracteres; sem legenda separada. |
| Depoimento | Foto | Uma | 20 MB. |
| Depoimento | Vídeo | Um | 500 MB; 300 segundos. |
| Depoimento | Áudio | Um | 50 MB; 300 segundos. |
| Momento | Fotos | Uma a dez | 20 MB por foto; uma legenda compartilhada. |
| Momento | Vídeo | Um | 500 MB; 60 segundos. |

Não misturar formatos, permitir áudio/texto em Momento ou confirmar parte de um conjunto. Legenda opcional de mídia tem 500 caracteres. Foto JPEG/PNG/HEIC/HEIF, vídeo MP4/MOV/WebM e áudio MP3/M4A/WAV/OGG/WebM exigem inspeção de container/codec e conteúdo decodificável; extensão/MIME declarado não são prova. Limites de decoder evitam dimensões/expansão abusivas sem inventar nova restrição de duração.

1. Antes do POST, navegador preserva chave e segredo separados; nada é gravado no banco ao apenas selecionar/gravar um conteúdo.
2. Records autoriza evento/janela por contrato e cria registro provisório, operação, sessão e manifesto de arquivos atomicamente. Conflito de unicidade consulta a operação existente e compara HMAC do payload; payload diferente com mesma chave falha. Só depois do commit emite autorizações S3.
3. Banco planeja chave temporária opaca antes do efeito externo. Upload/multipart acontece fora da transação; ID multipart é persistido e reconciliado. Repetem-se partes que falharam, sem novo registro lógico. Não persistir cada parte como uma nova mídia: S3 é fonte do multipart; ListParts reconstrói o progresso necessário.
4. Backend fixa versões recebidas, confere tamanho real, checksum, formato, duração e pertencimento à chave planejada. Usa LastModified/VersionId confiáveis do S3, não apenas horário do callback nem timestamp de cliente. O arquivo selecionado não pode ser trocado por PUT posterior; todo acesso/cópia usa versão fixada.
5. Promove/preserva original privado de forma recuperável. Se S3 concluir e o banco não confirmar, reconciliar objeto e intenção; não apagar um possível original por timeout. A evidência de prazo vem do recebimento original temporário; cópia posterior não muda esse fato.
6. Em transação local Records, conferir todas as linhas, contagem, posições, metadados e originais stored; fixar received_at/confirmed_at; concluir sessão/operação; criar jobs e Outbox. Só então responder confirmação ao convidado. `received_at` do grupo é o máximo dos arquivos; texto usa o recebimento do pedido.
7. Status/replay autorizado pode recuperar confirmação sem retornar conteúdo ou URL de leitura. Expirou o prazo de status: negar acesso ao convidado, conservar a unicidade mínima do registro confirmado. Não confundir resposta desconhecida com permissão para iniciar outro registro.

A sessão pode ficar validating após 13h quando há evidência de recebimento tempestivo. Expirar só após reconciliação; rejeição irreversível não admite confirmação tardia contraditória. `received_at/confirmed_at` confirmados são imutáveis. Falha de conversão muda disponibilidade, preservando recebimento e original.

### Armazenamento, variantes e processamento

Objetos são privados e possuem bucket, key opaca e version_id. Original não é URL, blob SQL ou ETag tratado como checksum. Objetos de tentativas diferentes usam prefixos distintos; uma tentativa atrasada não sobrescreve a saída selecionada. Metadados não guardam nome original do arquivo nem localização EXIF.

Fotos geram imagem JPEG até 2.048px no maior lado e thumbnail até 400px. Preservar imagens menores, proporção/orientação e remover localização das versões exibidas. Vídeo gera MP4/H.264/AAC até 1080p/30fps sem ampliar resolução ou taxa menor; áudio gera M4A/AAC. O job guarda versão e snapshot do perfil; bitrates e mídia vertical continuam pendentes de configuração comprovada.

Job possui tentativa local criada antes de chamar MediaConvert. Persistir request_key, identidade externa e prefixo de saída; `unknown` significa resultado incerto que exige reconciliação, não falha autorizando duplicação automática. Se o provedor oferecer idempotência, respeitar seu escopo/prazo comprovado; a tabela local não garante exatamente uma execução externa. Callback autenticado atualiza somente a tentativa correspondente e a versão ainda elegível. Fotos usam a mesma disciplina de posse, saídas e retries no consumidor .NET.

Registro com todas as variantes exigidas fica available. Enquanto houver trabalho, processing; falha definitiva fica processing_failed e gera alerta técnico. Uma foto pronta do conjunto pode ser exibida após a confirmação integral mesmo se outra ainda processa, com estado individual de arquivo. Nenhum desses estados permite exibição antes da abertura.

### Galeria e sigilo

Proposta para “primeiro envio ao último”: ordenar pelo início autorizado `(records.submitted_at, records.id, files.position)`, com posição 0 para texto. Assim o tamanho do upload/conversão não redefine a ordem. Somente registros com confirmed_at entram na consulta; cada foto é um item. Filtros usam category; até dez itens por página, cursor opaco validado com evento/filtro e autorização renovada.

A consulta expande arquivo por posição e preenche o restante da página se um grupo atravessar o limite. Não limitar dez registros antes de expandir dez fotos. Confirmação tardia pode inserir item numa posição anterior ao cursor: atualizar a galeria reinicia a leitura. Cursor estável não promete snapshot congelado entre requisições; o contrato deve explicitar essa semântica.

Com acesso bloqueado, interromper antes de consultar conteúdo, count, remetentes ou miniaturas. Não manter contadores públicos ou endpoint alternativo que revele recebimento. Capas e boas-vindas públicas ficam separadas dos objetos privados de lembranças. URLs CloudFront de leitura são geradas sob autorização, até dois minutos, e nunca persistidas/logadas.

### Notificações e auditoria

- O envio opcional de convite por e-mail é pedido explícito ao contrato de Notifications depois de o convite existir. Notifications grava destino criptografado, intenção e Outbox na própria transação. Se o fluxo interativo tiver duas chamadas internas, há dois commits locais: falha na segunda permite repetir o pedido de e-mail com request_key estável, sem recriar convite. Não transportar um destino que só existia na memória por uma Outbox contendo apenas IDs.
- Avisos de abertura chegam por mensagem e consultam o e-mail confirmado do responsável por contrato. deduplication_key é canônica: convite + request_key no pedido interativo, ou opening_dispatch no aviso. Replay compara destino com o hash_key_id original; outro destino com a mesma chave é conflito. Proteger o destinatário efetivamente usado em cada tentativa; não incluir e-mail/token no envelope de mensageria.
- Scheduler diário às 00h de Brasília faz Events encontrar eventos programados elegíveis e criar obrigações por responsável/versão ainda sem conclusão. Repetir o job reconcilia a mesma obrigação. Outbox → Notifications → SES → Outbox de conclusão → Events; sem escrita cruzada. Alteração de acesso cancela obrigações obsoletas. Um aviso aceito não é repetido apenas porque o ACK interno se perdeu.
- Antes de chamar SES, conferir convite/versão/destino por contrato e persistir tentativa protegida. Uma alteração concorrente entre leitura e chamada externa pode gerar aviso obsoleto; o link sempre reautoriza e não libera conteúdo. Não prometer atomicidade com SES.
- Accepted significa encaminhado ao SES, não entregue. Unknown é reconciliado; duplicata externa ainda é possível se o aceite teve resposta perdida. Feedback autenticado diferencia delivered/bounced/complained, sem reabrir envio lógico concluído.
- Recuperação/confirmação são fluxos Cognito; não criar envio paralelo de código em Notifications. A integração Cognito/SES deve ser medida para a meta de encaminhamento de recuperação. Alertas operacionais CloudWatch/SNS não são notificações de produto persistidas aqui.
- Auditoria local registra ator, ação, IDs/estados/datas permitidos, resultado e correlação por 180 dias. Mudança sensível bem-sucedida e auditoria entram na mesma transação; sem texto, nome de convidado, senha, mídia, e-mail ou URLs em logs/auditoria.

## Estados e transições

Catálogos completos estão no dicionário. Transições exigem estado anterior esperado, versão e/ou lock; CHECK sozinho não implementa uma máquina de estados.

| Agregado | Caminho normal | Falha / recuperação |
| --- | --- | --- |
| Conta | active → revocation_pending → active | disabled/anonymized conforme procedimento; nunca reativar sessões com security_version antigo. |
| Sessão | active → refreshing → active | revoked ou expired são terminais; refresh incerto exige reconciliação ou nova autenticação. |
| Convite | pending → accepted | pending → superseded/invalidated/expired; estados terminais não voltam a pending. |
| Capa | pending → validating → ready → superseded | rejected/superseded → deleting → deleted; capa atual não é elegível. |
| Registro | receiving → validating → confirmed → processing → available | Antes de confirmar: rejected; após confirmar: processing_failed, com reprocessamento controlado para processing/available. Texto pode ir de confirmado a disponível no mesmo commit. |
| Sessão de upload | authorized → receiving → validating → confirmed | rejected/expired somente após decisão irreversível reconciliada; confirmação não regride. |
| Operação idempotente | in_progress → confirmed | rejected/expired guardam resultado e unicidade; aposentar segredo não cria nova tentativa. |
| Objeto | planned → initializing/uploading → stored | Elegível: deleting → deleted; falha de chamada externa permanece reconciliável. Original confirmado não percorre limpeza comum. |
| Variante | pending → processing → ready | failed pode ter nova tentativa controlada; ready só troca de seleção por perfil/resultado autorizado. |
| Job de mídia | pending → running → completed | reconciling para resultado incerto; failed após limite configurado; reprocessar preserva histórico de tentativas. |
| Tentativa de mídia | created → submitted → running → succeeded | failed conhecido ou unknown incerto; sucesso tardio só aplicado se ainda elegível. |
| Obrigação de abertura | pending → requested → dispatched | cancelled por configuração obsoleta; requested aguarda confirmação eventual, não reenvio cego. |
| Notificação | pending → running → accepted | reconciling, failed ou cancelled; não reabrir accepted por retry interno. |
| Outbox / inbox | pending → publishing → published / processing → completed | Retry com mesmo message_id; failed/quarantined exige ação controlada, sem ignorar divergência de payload. |

## Transações, concorrência e idempotência

| Operação | Unidade atômica / proteção |
| --- | --- |
| Criar evento | Events: evento + slot 1 + auditoria; validação agregada do mínimo de membros. |
| Alterar estrutura/acesso | Events: lock do evento, avaliar janela anterior, revalidar travas, versão esperada, dados + auditoria + obrigações/Outbox necessárias. |
| Gerar/aceitar convite | Events: evento → convite → membro, ordem fixa; substituir pendente ou criar slot 2; constraints e auditoria. |
| Admitir/confirmar envio | Records: registro → operação/sessão → arquivos/objetos, ordem fixa; UNIQUE + fingerprint; confirmação integral + jobs + Outbox. |
| Aplicar primeiro registro | Events: inbox + evento travado + invalidação de convite + auditoria, sem transação com Records. |
| Consumir mensagem | Módulo dono: deduplicação + efeito local/intenção durável + Outbox, em um commit; ACK só depois. |
| Chamar provedor | Persistir intenção/tentativa antes; commit; chamar rede sem lock aberto; concluir condicionalmente pelo token de posse/versão ou reconciliar. |
| Revogar sessões | Identity: estado da conta + security_version + sessões locais + operação de reconciliação + auditoria. |

Padrão proposto: READ COMMITTED com locks explícitos nas decisões concorrentes, UNIQUEs e constraints agregadas diferíveis; retries de deadlock/conflito devem reutilizar chave lógica. Não elevar todas as operações a SERIALIZABLE nem usar lock de contagem de responsáveis: a capacidade vem dos slots únicos.

Claims de filas usam lote curto e `FOR UPDATE SKIP LOCKED`, lease_token e lease_until. Trabalho abandonado recebe token novo; conclusão exige token atual. Não tratar lease como fencing aceito pela AWS: para efeito externo usar tentativa identificável e reconciliação. [Documentação PostgreSQL](https://www.postgresql.org/docs/18/sql-select.html).

Inbox deduplica por consumidor/fonte/identidade, compara fingerprint e não usa receipt handle SQS como identidade de negócio. Após retenção técnica, UNIQUEs de registro, job, obrigação e notificação continuam impedindo repetição do efeito lógico. SQS Standard pode duplicar/reordenar; eventos atrasados não regridem estados finais. Sem garantia de exactly-once entre PostgreSQL, S3, Cognito, MediaConvert e SES.

Os [scripts correspondentes](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md) seguem existência + compatibilidade de estado, idempotência concorrente e rollback obrigatório dos [guidelines SQL](../guidelines/sql.md). Eles verificam o estado persistido, sem substituir autenticação, autorização ou prova de presença física no S3.

## Retenção, segurança e operação

| Dado | Retenção / descarte |
| --- | --- |
| Evento, membros e registros confirmados | Enquanto a plataforma for mantida; sem exclusão por interface. Procedimento de privacidade extraordinário ainda deve ser definido. |
| Originais confirmados | Preservados privados, inclusive após falha de conversão; versionamento não substitui estratégia de backup. |
| Incompletos/rejeitados | Elegibilidade de limpeza após 24h, com âncora P-05 e reconciliação; abortar multipart e remover as versões corretas. Nunca apagar dado confirmado por job atrasado. |
| Variantes/temporários | Temporário removível só após original durável ou rejeição reconciliada. Variante selecionada protegida; saídas órfãs exigem revalidação. |
| Credenciais locais protegidas | Expiração/revogação remove ciphertexts; prazo de replay/status não concede leitura de mídia. Identificar chave para rotação sem quebrar verificações ainda válidas. |
| Técnicos e auditoria | 180 dias; expurgar em lotes pequenos, respeitando pendências, tombstones de negócio e RESTRICT. Quarentena/DLQ não é sucesso descartável. |
| Backups | RDS com retenção de 35 dias e recuperação/versionamento S3 conforme HLD; validar restauração conjunta e impacto de dados já saneados. |

Privilégios de runtime sem DDL e sem alteração livre de auditoria; Liquibase com identidade própria. FKs são integridade, não autorização. Não declarar RLS configurada: isolamento será imposto em contratos/consultas autorizadas e testado; RLS adicional exigiria decisão própria. Chaves fora do repositório, envelopes criptográficos e rotação definidos com a infraestrutura.

Não há tabela para logs, métricas, rastros, status de saúde ou cache distribuído. A operação usa CloudWatch/X-Ray e alertas conforme HLD. Metas de 200 uploads simultâneos em dois eventos, p95 de API até 500ms, 95% das mídias prontas em até 5min, 99,9% de disponibilidade, RTO de 10min/RPO zero nos cenários acordados dependem de testes; índices/modelagem não comprovam essas metas.

## Rastreabilidade funcional

| Requisito | Cobertura no modelo / fronteira |
| --- | --- |
| FR-001 Cadastro, autenticação e sessão | accounts, sessions, oidc_requests; confirmação e credenciais Cognito. |
| FR-002 Conta e recuperação | account_operations, security_version, auditoria; código e senha no Cognito. |
| FR-003 Múltiplos eventos | events, event_members, covers; configuração isolada por evento. |
| FR-004 Segundo responsável | invitations, slots, notifications; validade/uso único/capacidade. |
| FR-005 Página e QR | código público, nome, capa e boas-vindas; QR derivado, sem registro de leitura. |
| FR-006 Janela e tolerância | snapshots em events/upload_sessions; evidências de recebimento, sem extensão por retry. |
| FR-007 Nome e concordância | consent_notices + snapshot em records; reutilização local no navegador. |
| FR-008 Depoimentos | category/format, message ou arquivo único. |
| FR-009 Momentos | photo de 1..10 ou video único; posições e confirmação integral. |
| FR-010 Captura/seleção | files guarda manifesto e metadados; captura/gravação é frontend. |
| FR-011 Limites/validação | tamanhos, duração, container/codec, checks locais e regra agregada de confirmação. |
| FR-012 Texto/legenda | message até 5.000, caption até 500, legenda compartilhada por grupo. |
| FR-013 Revisão/rascunho | Sem persistência antes de Enviar; proteção de navegação é frontend. |
| FR-014 Envio/progresso/retry | idempotent_operations, upload_sessions, files, storage_objects; multipart S3 e polling de 5s. |
| FR-015 Confirmação/restrições | confirmed_at durável, resultado seguro; convidado sem leitura posterior de mídia. |
| FR-016 Travamento | Outbox Records → inbox Events; locked_at e invalidação eventual. |
| FR-017 Abertura | opening_mode, opens_at, access_state/version, congelamento monotônico e obrigações. |
| FR-018 Bloqueio | Autorizar antes de consultar/retornar conteúdo, contagem, remetentes ou URLs. |
| FR-019 Galeria | registros confirmados e arquivos ordenados; filtros, cursor e dez itens; sem busca/download/exclusão. |
| FR-020 Processamento | jobs, attempts, variants, objetos versionados; original preservado. |
| FR-021 E-mails | opening_dispatches, notifications e attempts; recuperação Cognito; metas de 1min/5min devem ser medidas. |
| FR-022 Auditoria/alertas | audit_entries por módulo por 180 dias; erros sanitizados, Outbox/inbox e observabilidade externa. |

## Dependências antes da implementação

1. Alinhar os contratos HTTP/mensagens aos detalhamentos P-01 a P-07 materializados na base e provisionar RDS PostgreSQL 18 compatível com a validação local em 18.6.
2. Comprovar fluxos Cognito, normalização de e-mail, confirmação/troca, revogação global e proteção/rotação dos envelopes. Definir chaves e permissões por ambiente.
3. Fechar perfil MediaConvert/fotos, codecs reais, limites de decoder, parâmetros de lease/retry e reconciliação. Não transformar dimensões verticais/bitrates pendentes em promessa de suporte.
4. Detalhar limpeza de 24h e procedimento de privacidade fora da interface, inclusive backups; validar necessidade de armazenar cada dado pessoal.
5. Implementar mappings EF e o pipeline de implantação sobre os [scripts Liquibase existentes](https://github.com/diorgenesbackes/scrapbook-liquibase/blob/main/README.md), mantendo o backend sem migrations de aplicação.

Os cenários futuros de validação devem cobrir: terceiro responsável concorrente; convite vencido/substituído; limites exatos de 12h/13h; evento de 29/02; alteração de data sem destravar abertura; replay com payload divergente; foto faltante; callback tardio com upload no prazo; PUT posterior em chave temporária; sucesso externo com commit perdido; duplicatas/ordem de mensagens; limpeza concorrente; revogação entre réplicas; galeria bloqueada sem vazamento; atualização tardia em cursor; recuperação de backup. São critérios planejados, não testes executados nesta entrega documental.
