# Padrão de contratos para APIs síncronas

[Índice](README.md) · [Backend](backend.md) · [Frontend](frontend.md) · [Mensageria e mídias](mensageria.md) · [Observabilidade](observabilidade.md)

**Status:** padrão para os contratos a implementar. O prefixo e as regras de produto vêm do HLD/PRD; rotas detalhadas, payloads e códigos ilustrativos devem ser fechados na FDD/LLD e no OpenAPI.

## 1. Decisões de formato

| Tema | Padrão do Scrap Book |
| --- | --- |
| Transporte | HTTPS; REST/JSON UTF-8 para negócio. Arquivos seguem diretamente para S3. |
| Versionamento | Prefixo `/api/v1`; health operacional fora do contrato de negócio. |
| Request | Campos da operação, sem envelope universal `request` ou `data`. |
| Sucesso | Recurso ou objeto composto com propriedades de negócio nomeadas. |
| Coleção paginada | Cursor opaco; galeria com até 10 itens. |
| Sem representação | `204`, sem corpo. |
| Processamento posterior | Estado persistente explícito; `202` somente quando a operação foi aceita e permanece pendente. |
| Erro | `application/problem+json`, código estável, detalhe seguro em português e correlação. |
| Correlação | `X-Correlation-Id` na resposta; `traceId` e `correlationId` nos erros instrumentados. |

Não adicionar envelope com `success`, `statusCode`, `message` e `data` a toda resposta. Não envolver arquivos, redirecionamentos OIDC, health ou `204`.

A operação HTTP pode terminar enquanto a preparação da mídia continua. Distinguir **sessão autorizada**, **recebimento em validação**, **registro confirmado** e **mídia disponível**. Nenhum `2xx` intermediário deve ser apresentado como confirmação do registro.

## 2. Requests

### Métodos e entrada

| Método | Regra |
| --- | --- |
| GET | Consulta sem corpo ou alteração de negócio. |
| POST | Criação ou comando; `201` para recurso criado, `200/204` para resultado concluído, `202` para operação realmente pendente. |
| PUT | Substituição dos campos editáveis documentados; ausência tem semântica definida. |
| PATCH | Atualização parcial com campos permitidos e significado de ausência/`null` explícitos. |
| DELETE | Apenas quando houver requisito. A primeira versão não oferece exclusão de eventos ou registros pela interface. |

Usar recursos no plural, IDs opacos e `operationId` estável. Path identifica o recurso, query ajusta consulta, body contém entrada editável. O servidor obtém ator, vínculo, timestamps de recebimento e estados definitivos; não aceitar esses valores como verdade enviada pelo cliente.

### Convenções de dados

- JSON/query em `camelCase`; enums textuais documentados em `snake_case`.
- IDs de recursos derivados das PKs bigint são strings decimais, preservando o alcance no JavaScript; o cliente os trata como opacos. Identificadores técnicos, como correlação e mensagens, podem continuar UUIDs. Referências externas mantêm seu contrato próprio. Ver [SQL](sql.md#3-chaves-primárias-e-relacionamentos).
- Horários de negócio são locais de Brasília. Proposta para contratos novos: data `YYYY-MM-DD`, horário `HH:mm:ss` ou datetime local ISO sem offset, com a semântica fixa `America/Sao_Paulo` expressa no schema. Não permitir escolha de fuso.
- Instantes técnicos/externos usam ISO 8601 com offset explícito, preferencialmente `Z`; normalizar antes de comparar com um prazo de negócio. Não tratar datetime local como UTC.
- Coleções vazias são `[]`; booleanos são `true/false`; declarar nulabilidade, faixa, comprimento e normalização.
- Definir bytes e segundos para tamanho/duração técnicos. A conversão dos limites de MB do PRD para bytes e a unidade de contagem de caracteres devem ser uniformes entre frontend e backend e registradas na FDD.
- Rejeitar campos desconhecidos em novos comandos e usar allowlist de campos editáveis. Consumidores devem tolerar campos adicionais de resposta.
- Não aplicar trim ou normalização genérica a senhas, códigos e tokens.
- Contadores/versões fora do alcance inteiro seguro do JavaScript exigem representação textual ou limite documentado.

### Headers, limites e cache

| Header | Regra |
| --- | --- |
| `Content-Type` | `application/json` para JSON. Upload ao S3 segue o contrato de assinatura, separado do body da API. |
| `Accept` | JSON e Problem Details; formatos de mídia conforme a operação. |
| `X-Correlation-Id` | UUID opcional validado; devolver valor válido ou gerar outro. |
| `traceparent/tracestate` | Contexto de tracing, nunca credencial. |
| `Idempotency-Key` | Obrigatório ao iniciar a tentativa lógica de upload; outras operações declaram seu suporte. |
| Cookie / CSRF | Sessão intermediada pelo backend, com parâmetros e proteção definidos junto do OIDC. |
| `Cache-Control` | `no-store` para sessão, conteúdo privado, status de envio e respostas sensíveis; estáticos têm política própria. |

CORS deve usar origens explícitas e permitir/expor apenas os headers necessários, inclusive no S3. Credenciais, callback OIDC e CSRF exigem configuração compatível com os domínios escolhidos; CORS não autoriza acesso.

Cada endpoint deve declarar tamanho máximo de body e orçamento de execução, incluindo dependências. Os limites de mídia do produto se aplicam aos objetos enviados diretamente ao S3.

## 3. Respostas e paginação

Criação concluída retorna `201`, representação autorizada e `Location` quando existir recurso consultável. `204` não contém JSON. Operação pendente retorna estado e referência consultável restrita ao solicitante, sem prometer resultado concluído.

### Galeria

Formato proposto para uma página autorizada:

```json
{
  "items": [
    {
      "id": "101",
      "recordId": "100",
      "category": "moment",
      "mediaType": "photo",
      "senderName": "Convidado de exemplo",
      "caption": "Uma lembrança do evento.",
      "state": "available"
    }
  ],
  "nextCursor": null,
  "hasMore": false
}
```

O endpoint de galeria deve:

- Autorizar vínculo e abertura **antes** de consultar/devolver lembranças.
- Retornar até 10 itens e cursor opaco.
- Ordenar do primeiro envio ao último, com desempate único e posição estável dos arquivos no conjunto.
- Contar cada foto como item independente, preservando nome e legenda comuns.
- Vincular o cursor ao evento, filtro e ordenação; validar adulteração e não usá-lo como autorização.
- Aceitar filtros correspondentes a Depoimentos, Momentos e Tudo.
- Documentar a visibilidade de registros novos durante a navegação; cursor não promete snapshot por si só.

Não exigir `totalCount`. Antes da abertura, não devolver items, cursores, contagens, remetentes ou indicadores que revelem recebimento. A tela bloqueada usa apenas configuração permitida, data de abertura e referência de horário do servidor.

### Upload e confirmação

Rotas abaixo são exemplos de nomenclatura, não endpoints existentes:

| Operação ilustrativa | Resultado e restrição |
| --- | --- |
| `POST /api/v1/events/{eventId}/upload-sessions` | Autoriza tentativa lógica antes do encerramento, persiste arquivos esperados e prazo; exige chave idempotente. |
| `POST /api/v1/upload-sessions/{id}/completion` | Solicita validação da recepção; pode permanecer pendente. Repetição não cria registro novo. |
| `GET /api/v1/upload-sessions/{id}/status` | Estado mínimo da própria tentativa autorizada; polling a cada 5 segundos. Sem conteúdo ou URLs de leitura. |
| `POST /api/v1/events/{eventId}/media-authorizations` | Emite URL de até 2 minutos somente após nova verificação de sessão, vínculo e abertura. |

O contrato deve especificar como o convidado comprova autorização da própria tentativa sem login. O identificador da sessão não basta. Autorizações de escrita não concedem leitura; não expor listagem de registros ao convidado.

Confirmar apenas quando arquivos e dados necessários estiverem persistidos. Uma confirmação com mídia ainda em processamento é válida; uma autorização de upload não é confirmação. Depoimentos em texto seguem o mesmo princípio de persistência, sem criar arquivo fictício.

## 4. Erros e catálogo inicial

Adotar Problem Details, com status HTTP coerente e extensões do projeto:

| Campo | Regra |
| --- | --- |
| `type/title/status` | Tipo e título estáveis; inicialmente `about:blank` e título correspondente ao status; inteiro igual ao HTTP. |
| `detail` | Mensagem segura em português, sem decisão do cliente baseada no texto. |
| `code` | Código estável `SCREAMING_SNAKE_CASE`. |
| `errors` | Validação por caminho camelCase; `$` para erro geral. |
| `traceId` | ID W3C do trace, quando instrumentado. |
| `correlationId` | UUID devolvido em `X-Correlation-Id`. |
| `instance` | Opcional; ocorrência opaca, sem query, token ou URL utilizável. |

```json
{
  "type": "about:blank",
  "title": "Conflict",
  "status": 409,
  "detail": "O prazo para concluir este envio terminou.",
  "code": "UPLOAD_COMPLETION_DEADLINE_EXCEEDED",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "correlationId": "97dfba56-bdf4-49ba-8f64-3cf9e14c134a"
}
```

Catálogo proposto, a detalhar por endpoint:

| HTTP | Código | Situação |
| --- | --- | --- |
| `400` | `REQUEST_VALIDATION_FAILED` | JSON, campos, limites declarados ou cursor inválidos. |
| `401` | `IDENTITY_AUTHENTICATION_REQUIRED` | Sessão ausente/expirada em operação protegida. |
| `403` | `RESOURCE_ACCESS_DENIED` | Sem permissão; usar `404` quando a política exigir ocultar existência. |
| `403` | `EVENT_CONTENT_LOCKED` | Responsável vinculado, mas conteúdo ainda bloqueado. Sem metadados de recebimento. |
| `404` | `RESOURCE_NOT_FOUND` | Recurso inexistente ou oculto conforme política. |
| `409` | `EVENT_LOCKED` | Alteração incompatível com o travamento persistido. |
| `409` | `EVENT_VERSION_CONFLICT` | Atualização concorrente não pode sobrescrever a anterior. |
| `409` | `UPLOAD_WINDOW_CLOSED` | Novo envio fora da janela. |
| `409` | `UPLOAD_COMPLETION_DEADLINE_EXCEEDED` | Conclusão fora da tolerância. |
| `409` | `INVITATION_UNAVAILABLE` | Expirado, usado, substituído ou impedido pelo estado do evento. |
| `409` | `IDEMPOTENCY_KEY_REUSED` | Mesma chave com entrada diferente. |
| `409` | `IDEMPOTENCY_REQUEST_IN_PROGRESS` | Tentativa lógica já está em andamento. |
| `413` | `REQUEST_TOO_LARGE` | Body da API excede o limite. Arquivo inválido pode ser rejeitado posteriormente no status. |
| `415` | `UNSUPPORTED_MEDIA_TYPE` | Tipo de body não suportado. |
| `429` | `REQUEST_RATE_LIMITED` | Proteção de entrada/login; calibrar para Wi-Fi compartilhado. |
| `500` | `UNEXPECTED_ERROR` | Falha interna sanitizada. |
| `503` | `DEPENDENCY_UNAVAILABLE` | Indisponibilidade reconhecida de dependência. |

Rejeição assíncrona de arquivo precisa de código/motivo estável no status; não inventar um HTTP retroativo para a chamada já encerrada. Erros externos de ALB/WAF/S3 podem não ser JSON; o cliente deve oferecer fallback seguro. `Retry-After` só quando houver prazo conhecido e não torna uma escrita repetível.

## 5. Concorrência e repetição

Para comandos com idempotência, definir escopo, obrigatoriedade, retenção e fingerprint canônico. Em upload, a identidade da tentativa deve sobreviver a uma resposta perdida e funcionar sem autenticação de conta:

1. Mesma tentativa autorizada, chave e conteúdo: recuperar o resultado persistido, sem novo registro.
2. Mesma chave e conteúdo diferente: conflito.
3. Operação em andamento: devolver estado/conflito documentado, sem iniciar outra execução.
4. Resposta perdida: consultar ou repetir a mesma tentativa, respeitando autorização e prazos.

A retenção deve cobrir a recuperação prevista e a tolerância de envio; o valor ainda será definido. Chave idempotente não é credencial. Reautorizar o replay, sem conceder leitura do registro já enviado nem exigir que a janela continue aberta para consultar um sucesso anterior.

Escritas de evento/convite precisam de transação e atualização condicional ou locks. Versão esperada, se exposta, deve ser verificada atomicamente.

## 6. Registro mínimo e compatibilidade

Cada endpoint deve documentar: método/rota/`operationId`, finalidade, responsabilidade/serviço, acesso público ou autenticado, autorização por evento/data, CSRF, schemas, headers, cache, limites, tempos, sucesso, erros, estados, cursor, concorrência e idempotência. Definir também operação de telemetria e classificação dos dados.

Mudanças incompatíveis exigem versão ou transição explícita. Analisar novos valores de enum, propriedades obrigatórias e mensagens ainda em processamento; adicionar campo opcional não autoriza vazar metadados bloqueados.

## 7. Correlação e aceite

Aceitar um único `X-Correlation-Id` UUID canônico. Valor ausente, inválido ou múltiplo deve ser substituído; não refletir texto arbitrário. Devolver nos sucessos/erros alcançados pela API. `traceId`, `requestId` e `correlationId` têm funções distintas, conforme [Observabilidade](observabilidade.md).

- [ ] OpenAPI corresponde ao JSON, headers, limites e estados reais.
- [ ] API distingue autorização, recepção, confirmação e processamento.
- [ ] Cursor, ordem, filtros e coleção vazia foram verificados.
- [ ] Não há contagem, remetente, conteúdo ou URL antes da abertura.
- [ ] Repetição, resposta perdida, concorrência e prazos foram testados.
- [ ] Erros são sanitizados e correlacionados; cliente tolera falha não JSON.
- [ ] Convidado consulta apenas sua tentativa e não acessa a lembrança confirmada.
