### FDD: Privacidade pública do evento

Versão: 0.1
Data: 2026-11-13
Responsável: Diórgenes Backes

[História PP-STORY-006](https://trello.com/c/IV6uDrPq) · [PRD](../../PRD.md) · [HLD](../../HLD.md)

---

### 1. Contexto e motivação técnica

Consolidar a privacidade das histórias PP-001 a PP-005. Convidado, responsável vinculado e terceiro recebem a mesma apresentação pública. Conhecer o código compartilhável não concede acesso administrativo, a registros ou a mídia privada. A autenticação real permanece posterior; contas sintéticas explícitas permitem testar o limite de autorização local.

---

### 2. Objetivos técnicos

- Conferir lista exata de campos e projeção SQL sem identidade/contribuições.
- Manter apresentação e compartilhamento independentes da existência de registros e da abertura da galeria.
- Impedir persistência, prefetch e exposição de dados privados no cliente público.
- Remover URLs utilizáveis dos sinais locais e impedir sua propagação por Referer.

---

### 3. Escopo e exclusões

**Incluído**

- Revisão de API, SQL, cliente, proxy, cache, erros e exportações digitais existentes.
- Fixtures explícitas de registros privados somente no laboratório descartável.
- Regressões de autorização, tráfego/DOM/storage, spans e dimensões de métricas.

**Excluído**

- Implementar galeria, mídia, captura/envio, Cognito, upload de capa, AWS ou migrations.
- Alegar validação de aparelhos/papel, leitor real, produção ou coletor externo.

---

### 4. Fluxos detalhados e diagramas

**Fluxo principal**

1. SPA abre /e/:code sem carregar gestão ou enviar cookies na consulta pública.
2. API valida código, projeta somente code/name/welcome_message/starts_at e calcula janela com uma referência do servidor.
3. DTO público puro é serializado; browser constrói novo modelo com campos permitidos e capa local conhecida.
4. Nome e mensagem são texto; QR/link e material exportado contêm apenas apresentação compartilhável.

**Fluxos alternativos e exceções**

- Comparar mesmo evento antes/depois de inserir registros privados, respeitando apenas a passagem de serverTime.
- Repetir nos modos de galeria immediate/scheduled, com visitante/conta vinculada/terceiro.
- Navegar de gestão para público e entre dois eventos, sem reaproveitar estado anterior.
- Falhas mantêm mensagens sanitizadas de PP-005. Redirecionamento HTTP de leitura é recusado; repetição permanece manual.

---

### 5. Contratos públicos (assinaturas, endpoints, headers, exemplos)

**Apresentação**

- GET /api/v1/public/events/{code}, sem sessão obrigatória, orçamento API 5s/provider 4s/browser 6s.
- Raiz exata: code, name, effectiveWelcomeMessage, cover, publicUrl, serverTime, submissionWindow.
- cover exato: kind=default e assetKey=event-cover-default-v1. Não recebe URL arbitrária ou referência de contribuição.
- submissionWindow exato: state, opensAt, closesAt, timeZone, revalidateAfterMs, com semântica de PP-004.
- publicUrl: origem configurada e /e/:code; código base64url canônico, distinto da PK.
- Headers de aplicação: Content-Type JSON, Cache-Control no-store, X-Correlation-Id UUID; Referrer-Policy no-referrer e X-Content-Type-Options nosniff. CORS somente para origens locais configuradas. Headers técnicos Date/Server/Connection/Content-Length e tracing padrão não são dados de evento; nenhum Set-Cookie, Location, ETag ou Last-Modified decorre da consulta.
- HTML da SPA/fallback: no-store/no-referrer; estáticos públicos podem ser cacheados sem autorização de conteúdo privado.

**Exemplo de requisição**

```http
GET /api/v1/public/events/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Accept: application/json
```

**Exemplo de resposta**

```json
{"code":"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA","name":"Encontro","effectiveWelcomeMessage":"Bem-vindos!","cover":{"kind":"default","assetKey":"event-cover-default-v1"},"publicUrl":null,"serverTime":"2030-06-15T19:00:00.000Z","submissionWindow":{"state":"open","opensAt":"2030-06-15T16:00:00","closesAt":"2030-06-16T12:00:00","timeZone":"America/Sao_Paulo","revalidateAfterMs":30000}}
```

Proibidos: id/PK, version/accessVersion, vínculos/permissões, first_record_id, contagem/estado/conteúdo de registros, nome de remetente, consentimento, chaves de storage e URLs privadas/assinadas. Capa padrão é pública; código/link/QR são compartilháveis intencionalmente; conteúdo e identidade dos convidados são privados. PNG/print autorizados incluem somente nome público, link e QR; filename usa código público, sem nome de convidado/ID privado.

---

### 6. Erros, exceções e fallback

404 para link inválido/ausente; 400 para query extra; 401 para operação privada sem ator; 404 para recurso alheio; 503 reconhecido e 500 inesperado sanitizados conforme PP-005. Não carregar dados privados para decidir mensagens ou disponibilidade. Cache, estado de erro e material de QR não dependem da existência de registros. GETs não seguem redirects nem repetem falhas automaticamente.

---

### 7. Observabilidade

**Métricas**

Instrumentos existentes ScrapBook.Events: operação, método, status, resultado, ambiente e dependência, sem IDs/nomes/código/URLs. Conferir medições reais via MeterListener local.

**Logs**

JSON da API usa rota nominal e correlação UUID. EF sem SQL/sensitive data em logs. Nginx emite apenas componente, operação/método categorizados, status e duração; log de erro padrão do servidor é suprimido porque não admite redação da URI. Falhas continuam visíveis pelo status no log seguro e pelo diagnóstico da API.

**Tracing**

Conferir spans próprios ScrapBook.Events e dependências, tags permitidas e ausência de corpo/código/URLs. Nenhum exportador OpenTelemetry/X-Ray existe; sua configuração futura deve repetir a redação dos sinais automáticos de HTTP/SQL e não exportar baggage arbitrário.

**Dashboards e alertas**

Revisão local de status/duração. Coleta remota, WAF/ALB/Amplify, retenção e painéis permanecem configuração operacional posterior.

---

### 8. Dependências e compatibilidade

| Componente | Versão mínima | Observações |
| --- | --- | --- |
| API | SDK 10.0.401/runtime 10.0.12 | Clean Architecture existente. |
| UI | Node 26.13.0/npm 11.20.0 | Sem pacote novo. |
| PostgreSQL/Liquibase | 18.6/4.33.0 | Mesmo histórico de 47 changesets; sem DDL novo. |
| Histórias | PP-001 a PP-005 | PP-005 incorporada somente à feature PP-006, criada da main após pull. |

**Garantias de compatibilidade**

Payload não muda. Promover PP-005 antes de PP-006 em cada componente. Publicar API/proxy e depois UI; reverter UI e API/proxy sem apagar dados. Histórias anteriores e evidências permanecem históricas; QA humano pendente continua separado.

---

### 9. Critérios de aceite técnicos

- AC01: lista exata da raiz e objetos aninhados nos três estados temporais; SQL sem leitura de registros/identidades.
- AC02: DOM, rede, estado projetado, storage, cache e navegação públicos sem dados administrativos/privados.
- AC03: igualdade do contrato público com/sem ator e com/sem registros, nos modos de galeria existentes.
- AC04: PK/código/token não substituem vínculo/ator em operações privadas existentes; mídia/galeria futuras continuam sem implementação nesta entrega.
- AC05: logs, spans próprios, métricas e exportações digitais sem dados privados ou URLs utilizáveis de diagnóstico.
- Build/unitários e suítes HTTP/browser pertinentes aprovados, falhas e cenários não executados documentados.

---

### 10. Riscos e mitigação

### Instrumentação automática futura

- **Probabilidade:** média.
- **Impacto:** exportar URL/SQL ou baggage não auditados.
- **Mitigação:** manter exportador desabilitado, validar tags e configurar redação antes de habilitá-lo.
- **Plano de contingência:** suspender coleta afetada e remover dados conforme procedimento operacional.

### Falsa equivalência com funcionalidades posteriores

- **Probabilidade:** média.
- **Impacto:** inferir que mídia privada, Cognito ou galeria real foram validados.
- **Mitigação:** testar contratos e autorização existentes; registrar ausência de rotas e limites da fixture.
- **Plano de contingência:** exigir ensaios próprios quando cada funcionalidade for implementada.
