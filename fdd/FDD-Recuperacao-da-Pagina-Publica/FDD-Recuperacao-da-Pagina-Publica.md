### FDD: Links inválidos e recuperação da página pública

Versão: 0.1
Data: 2026-11-13
Responsável: equipe Scrap Book
Situação: implementada/validada localmente em feature/PP-STORY-005, sem merge/deploy. [Relatório e evidências](relatorio.md).

### 1. Contexto e motivação técnica

[PP-STORY-005](https://trello.com/c/QhI07Cia), [PRD FR-005](../../PRD.md#fr-005-página-dos-convidados-e-qr-code), [HLD](../../HLD.md#interfaces-públicas), [S35](../../visual-identity/screens/S35.md). Completar erros/recuperação de apresentação e compartilhamento, preservando os contratos e o isolamento existentes.

Stack parada; PP-STORY-003/004 integradas por solicitação expressa em cinco mains. Todas as branches anteriores auditadas como ancestrais de main. Pull executado nos seis repositórios após os pushes, antes da criação/codificação de feature/PP-STORY-005. Evidências de merge/base no ciclo desta história; QA anterior de aparelhos/papel e leitor real continua pendente.

### 2. Objetivos técnicos

- Distinguir loading, não encontrado, rede, timeout, limitação e serviço indisponível com texto seguro e ação real.
- Retry de GET na mesma rota, sem duplicação, refresh infinito ou restauração de disponibilidade antiga.
- API retorna Problem Details/correlação segura nos erros que alcançam seu pipeline; UI tolera respostas não JSON de proxy/WAF.

### 3. Escopo e exclusões

Inclui caminhos públicos vazios/malformados, erro sanitizado, correlação, timeout/cancelamento de leitura e retry acessível em S16/S15. Reutiliza fallbacks de capa, clipboard, QR, PNG e print. Sem autenticação, suporte por mensagem, captura/upload, galeria, DDL ou AWS. Somente a nova feature é desenvolvida; seu merge não é automático.

### 4. Fluxos detalhados e diagramas

Consultar por GET sem cache; código público não carrega cookies, compartilhamento privado preserva credenciais/autorização existentes. UI mede timeout local de 6s, com cancelamento vinculado à navegação; API mantém orçamento total de 5s e provider de 4s. O segundo adicional permite transporte/serialização, sem estender prazo de envio.

Código malformado é recusado antes da persistência; código canônico ausente consulta o índice. Ambos retornam a mesma representação pública 404 RESOURCE_NOT_FOUND, sem indicar outro evento. GET vazio/catch-all inválido também recebe o Problem Details; rota simples válida tem precedência. O shell guest existente mantém links inválidos fora da navegação de gestão.

Na falha, classificar por status/tipo, nunca renderizar detail/message brutos. Rede tem orientação de conexão; timeout tem orientação de repetir; 502/503/504 indicam indisponibilidade; 429 pede espera; 500/contrato inválido são inesperados; 401/403 não sugerem login no público. 404 retira dados anteriores. Falhas transitórias preservam apresentação/link/QR conhecidos, retirando a afirmação de disponibilidade.

Botão de leitura permanece no DOM e mantém foco durante a tentativa, com aria-disabled/guard síncrono. Sucesso troca a ação para Atualizar evento. Não há retry automático de erro; foco/online/retorno da aba podem consultar, respeitando cooldown. Ocultar uma aba com evento conhecido preserva o erro e a indicação visual de espera. Retry-After válido usa delta-seconds (até nove dígitos) ou data HTTP relativa ao Date do servidor, nunca ao relógio do aparelho; ausente/inválido em 429 usa 30s. Ao acabar a espera, habilitar ação, sem disparar consulta. Esperas longas são agendadas em blocos dentro do limite dos timers.

Revalidação regular da janela continua em até 30s/borda. Lease consumido no transporte permite uma única consulta adicional após 250ms; duas respostas consecutivas vencidas passam a erro seguro, sem loop. Troca de rota/unmount/ocultação cancelam consultas antigas; aborto de navegação não vira falha visível. Recuperação usa a janela nova do servidor.

### 5. Contratos públicos (assinaturas, endpoints, headers, exemplos)

GET /api/v1/public/events/{code} preserva os sete campos de sucesso da PP-STORY-004. Acrescenta handlers GET vazio e catch-all para erro; não há método de escrita ou nova sessão. GET privado /api/v1/events/{id}/share mantém projeção code/name/publicUrl e vínculo obrigatório.

| Situação | API/cliente |
| --- | --- |
| Código vazio/malformado/ausente | 404 RESOURCE_NOT_FOUND, application/problem+json, no-store e UUID em X-Correlation-Id/body. |
| Query indevida | 400 REQUEST_VALIDATION_FAILED. |
| Provider/conexão/budget reconhecido | 503 DEPENDENCY_UNAVAILABLE, sem erro SQL ou senha. |
| Inesperado | 500 UNEXPECTED_ERROR, mensagem fixa. |
| Conexão do navegador falhou | ReadFailure network, sem detalhe técnico. |
| Limite de 6s sem resposta | ReadFailure timeout; cancela fetch, inclusive se o transport mock ignorar o sinal. |
| Proxy/WAF 429/5xx sem JSON | Classificação pelo status; não interpretar HTML. |
| Payload de sucesso inválido | ReadFailure unexpected; não publicar dados de outro evento. |

Exemplo de erro:
```json
{"type":"about:blank","title":"Not Found","status":404,"detail":"Evento não encontrado.","code":"RESOURCE_NOT_FOUND","correlationId":"11111111-2222-3333-4444-555555555555"}
```

Kestrel/Nginx/WAF podem recusar URI/encoding/tamanho antes da aplicação. Esses erros não têm garantia de Problem Details/correlação da API; o frontend continua útil sem JSON em consultas. Para navegação de documento recusada pelo Nginx local, error_page 400 serve internamente invalid-link.html: mensagem guest em português, sem JavaScript, URI refletida ou redirecionamento, status 400 e no-store. proxy_intercept_errors permanece off, preservando Problem Details da API. Esse fallback não comprova configuração de WAF/Amplify futuros. Em chamadas entre origens, o edge precisa expor Retry-After/Date para a interpretação de datas; a configuração local usa proxy de mesma origem.

### 6. Erros, exceções e fallback

Sem retry de POST/PUT ou simulação de envio. Timeout de leitura tem no máximo uma tentativa ativa; duplo clique/foco/online não sobrepõe. Somente UUID canônico é mostrado como referência selecionável pelo teclado; URL/código de evento e mensagens livres não entram no componente de erro. Campos novos do contrato continuam tolerados por leitores existentes.

Erro de capa mantém nome/mensagem. Falha de geração QR mantém link/cópia e oferece geração manual. Falha de clipboard oferece seleção; PNG inválido não inicia download; print não afirma saída de papel. 404/401/403 de compartilhamento retiram dados anteriores, com retorno contextual ao evento.

### 7. Observabilidade

Reutilizar events.public_get/events.share_get e dependências SQL, com rota nominal, operação, status, duração, classe de exceção e correlação. Handlers públicos inválidos entram na mesma operação nominal. Sem body, URI completa, código, nome ou mensagem de exceção nos logs/labels. Cliente abandonado registra client_aborted sem Problem Details de negócio. Sem painel/SLA AWS novo declarado.

### 8. Dependências e compatibilidade

Mesmas versões fixadas .NET/React/TypeScript/Playwright; nenhum pacote novo. API/HTTP em API, Domain/Application continuam puros e Services exclusivos de Application. ReadFailure/readJson/ReadControls são fronteiras de leitura da UI; requestJson mantém o comportamento das mutações existentes, acrescentando apenas leitura de Retry-After.

Nova UI funciona com sucesso da API anterior; publicar handlers de erro antes para garantir o formato nos caminhos vazios. Rollback: UI, depois API; banco preservado. Nenhuma migration ou configuração de relógio/segredo nova.

### 9. Critérios de aceite técnicos

Mapeamentos sanitizados/correlação/cancelamento em xUnit; casos UI offline, 404/429/5xx/não JSON, timeout, Retry-After, foco, duplicação e respostas tardias em Vitest. HTTP real testa vazio/malformado/longo/encoding/ausente, banco indisponível e lock/timeout com recuperação. Navegador confirma retry sem reload/login, janela nova, alternativas de imagem/QR e teclado. Emulação/árvore acessível não são teste com leitor/aparelho real.

Testes manuais usam exclusivamente Compose/banco/volumes aleatórios; mesmos 47 changesets. Acionar regressões pertinentes e medir página no perfil 4G existente e carga pública de leitura; não extrapolar para produção. Registrar resultados/falhas e transições reais por PP-005-NNN.

### 10. Riscos e mitigação

Erros do edge sem JSON: classificar por status e omitir correlação inválida. Aba suspensa/timeout: abortar trabalho antigo, ignorar resultado tardio e consultar novamente sem usar disponibilidade antiga. Corrida de retry: guarda síncrona e cooldown monotônico. Referência de atendimento: UUID selecionável, sem envio de mensagem ou diagnóstico privado. Teste de banco/lock: somente recursos descartáveis, restauração/cleanup no finally. Pendências humanas anteriores permanecem explícitas.
