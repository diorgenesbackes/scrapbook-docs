### FDD: Disponibilidade dos envios

Versão: 0.1
Data: 2026-11-13
Responsável: equipe Scrap Book
Situação: implementada/validada localmente em feature/PP-STORY-004, sem merge/deploy. [Relatório e evidências](relatorio.md).

---

### 1. Contexto e motivação técnica

[PP-STORY-004](https://trello.com/c/lDmeo2EA), [PRD FR-006](../../PRD.md#fr-006-janela-de-envio-e-tolerância-para-conclusão), [HLD](../../HLD.md#interfaces-públicas) e [S16](../../visual-identity/screens/S16.md). Informar participação usando o relógio do servidor, independentemente da abertura da galeria. Reutilizar EventPolicy.Window e BusinessClock; não criar uma segunda regra no navegador.

feature/PP-STORY-004 nasce de main e inclui PP-STORY-003 somente na feature para preservar apresentação/link/QR. Promover PP-STORY-003 primeiro. Nenhum merge em main autorizado nesta rodada. Pendências PP-003-034 de aparelhos/papel e PP-002-029 de leitor real permanecem separadas.

---

### 2. Objetivos técnicos

- Classificar not_open/open/closed por uma única amostra UTC por resposta, convertida para America/Sao_Paulo.
- Abrir em startsAt menos duas horas, inclusive; encerrar ao meio-dia seguinte, exclusive para novas tentativas.
- Revalidar no máximo em 30s ou na próxima borda, o que ocorrer primeiro; nunca avançar estado localmente por relógio civil.
- Preservar apresentação/link/QR durante consulta ou falha transitória, retirando a afirmação de disponibilidade antiga.

---

### 3. Escopo e exclusões

**Incluído**
- Extensão aditiva do GET público com serverTime e submissionWindow, política pura, projeção mínima, OpenAPI e UI.
- Referência monotônica do navegador, foco/visibilidade/conectividade, timers com cleanup e estados acessíveis.
- Testes temporais unitários, HTTP com relógio substituído exclusivamente no host de testes, navegador, regressão e carga/performance local.

**Excluído**
- Captura, seleção, identificação/concordância, sessão/autorização de upload, mídia, galeria, login e AWS.
- Nenhum endpoint/configuração de produção altera o relógio. Sem DDL ou mudança nas 47 migrations.
- Capturar/selecionar não inicia envio; tolerância até 13h exige tentativa autorizada antes de 12h e será implementada pelo backend de uploads.

---

### 4. Fluxos detalhados e diagramas

**Fluxo principal**
- Consultar o código público existente, projetando code/name/welcome_message/starts_at sem identidade, registros ou escrita.
- Após a consulta, obter clock.UtcNow uma única vez, calcular marcos por EventPolicy.Window e classificar usando BusinessClock.ToCivil.
- Responder no-store. Estado: agora < abertura = not_open; abertura ≤ agora < encerramento = open; agora ≥ encerramento = closed, inclusive 12h–13h e depois de 13h.
- UI mede duração completa da requisição com performance.now, subtrai de revalidateAfterMs e agenda nova leitura. Sem Date.now ou fuso local como decisão.
- Enquanto revalida, não anunciar open antigo. Nome/capa/link/QR podem continuar visíveis; resultado atual determina o próximo estado.

**Fluxos alternativos e exceções**
- Documento oculto cancela consulta/timer e invalida a disponibilidade; retorno visível/foco/online consulta novamente, coalescendo chamadas.
- Resposta de outra rota, abortada ou mais antiga que a referência já aceita não restaura disponibilidade. Mesmo serverTime é permitido no relógio congelado de testes.
- Prazo consumido no transporte não autoriza exibir open: manter estado a confirmar e reconsultar após 250ms. Esse caso não é retry de erro HTTP nem de escrita.
- Falha de revalidação preserva apresentação conhecida, mostra disponibilidade não confirmada e retry manual/foco/online. 404 retira os dados anteriores.
- Alterações permitidas na gestão refletem na consulta seguinte, em até 30s em aba ativa; estado nunca é autorização de escrita. Backend de envios validará tudo novamente.
- Horários exibidos são civis de Brasília; serverTime é instante UTC. Não fixar offset -03:00 nem reinterpretar a data no fuso do aparelho.

---

### 5. Contratos públicos (assinaturas, endpoints, headers, exemplos)

**Apresentação pública**
- Tipo: endpoint; GET /api/v1/public/events/{code}, anônimo, sem query/body/cookie obrigatório.
- Campos anteriores preservados: code/name/effectiveWelcomeMessage/cover/publicUrl. Acrescentar serverTime UTC e submissionWindow obrigatório.
- Janela: state = not_open|open|closed; opensAt/closesAt = yyyy-MM-ddTHH:mm:ss civil sem offset; timeZone fixo America/Sao_Paulo; revalidateAfterMs inteiro 0–30000.
- Prazo calculado por floor da distância civil até a próxima borda, limitado a 30s; zero exige nova leitura sem anunciar disponibilidade. Closed também revalida em 30s para detectar eventual alteração permitida do evento.
- Sem completionDeadlineAt, openingMode, openingDate, permissões, contagens, IDs ou recebimentos na resposta pública.
- 200 válido; 404 código inválido/ausente; 400 query indevida; 503 dependência; 500 inesperado sanitizado. Cache-Control: no-store, X-Correlation-Id e orçamento total de 5s existentes.

**Exemplo de requisição**
```http
GET /api/v1/public/events/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

**Exemplo dos campos adicionados**
```json
{
  "serverTime": "2030-06-15T19:00:00.000Z",
  "submissionWindow": {
    "state": "open",
    "opensAt": "2030-06-15T16:00:00",
    "closesAt": "2030-06-16T12:00:00",
    "timeZone": "America/Sao_Paulo",
    "revalidateAfterMs": 30000
  }
}
```

Janela do exemplo começa às 16h de Brasília. Às 18:59:59.999Z, not_open com prazo 1ms; às 19:00Z, open; em 16/06 às 15:00Z, closed. Às 16:00Z (13h local), continua closed. Campos são informativos, não uma sessão de envio.

---

### 6. Erros, exceções e fallback

| Condição | Tratamento |
| --- | --- |
| JSON/enum/data/zona/prazo inválidos | Recusar o contrato; disponibilidade não confirmada. |
| Resposta atrasada/antiga ou aba oculta | Não inferir disponibilidade; cancelar/descartar e consultar quando apropriado. |
| Rede/503/revalidação falhou | Preservar apresentação conhecida, erro textual e retry seguro de GET. |
| 404 | Estado sem dados de evento, sem redirecionar para login. |
| Serviço parado/retomado | Recalcular pela data do evento e horário do servidor; jamais estender prazos. |

Sem retry de escrita, fallback para hora do aparelho, cache persistido ou permissão por botão. O temporizador só pede uma nova resposta; nunca transforma not_open em open ou open em closed por conta própria.

---

### 7. Observabilidade

**Métricas**
- Reutilizar events.public_get, duração/resultado/erros e events.sql.public_get existentes. Sem código, horários do evento ou estado particular como dimensão de alta cardinalidade.

**Logs**
- JSON/correlação/rota nominal existentes; não logar corpo, código público, nome ou URL utilizável.

**Tracing**
- Spans existentes de consulta pública e PostgreSQL; amostragem conforme ambiente, sem eventos por tick do relógio.

**Dashboards e alertas**
- Painéis remotos permanecem na implantação de observabilidade; esta história valida medições locais e privacidade dos logs, sem declarar painel AWS criado.

---

### 8. Dependências e compatibilidade

| Componente | Versão mínima | Observações |
| --- | --- | --- |
| Backend | .NET 10.0.401 SDK / runtime 10.0.12 | Sem pacote novo; BusinessClock/TimeProvider substituíveis em testes. |
| UI | React 19.3.1 / TypeScript 7.0.2 | Dependências fixadas existentes; sem biblioteca de relógio. |
| Testes | Playwright 1.65.0 / Python 3.13.15 | Acionamento manual, loopback e banco descartável. |
| PP-STORY-003 | Revisões registradas na base | Link/QR preservados; não integrar main implicitamente. |

**Garantias de compatibilidade**
- API aditiva, clientes anteriores ignoram campos novos. Publicar API antes da UI que exige o novo contrato; reverter UI antes da API.
- Domain/Application puros; política/model/enum/response em Domain, serviço em Application, EF em Infrastructure e HTTP/OpenAPI na API.
- Sem migração, cache persistido, dado privado novo ou mudança da abertura da galeria.

---

### 9. Critérios de aceite técnicos

- Antes/no/depois da abertura e encerramento, 12h–13h, meia-noite e viradas de mês/ano/bissexto verificados com relógio controlado.
- UI com fuso/relógio divergentes mantém Brasília; timers/foco/online, alteração de evento, resposta antiga, troca de rota e falha não anunciam open vencido.
- HTTP real com PostgreSQL e provider exclusivo de testes verifica bordas, serverTime único, no-store e independência da galeria, sem endpoint para alterar hora.
- Navegador mantém apresentação/QR ao atualizar ou falhar; anúncios não movem foco. Nenhum botão de captura sem destino.
- Regressões e carga de leitura local: zero erros, p95 até 500ms; página com QR pronta até 3s no perfil 4G declarado.
- Evidências e transições reais por PP-004-NNN; testes não executados ficam explicitamente pendentes.

---

### 10. Riscos e mitigação

### Relógio, transporte e aba suspensa

- **Probabilidade:** média.
- **Impacto:** disponibilidade antiga parecer atual.
- **Mitigação:** prazo curto vindo do servidor, duração monotônica completa descontada, invalidação em revalidação/ocultação, estado seguro na falha.
- **Plano de contingência:** consultar novamente; autoridade de escrita continuará sendo o backend de upload futuro.

### Confusão entre novos envios, tolerância e galeria

- **Probabilidade:** média.
- **Impacto:** abrir acesso ou prometer prazo incorreto.
- **Mitigação:** política não recebe modo/data da galeria, não expõe prazo de conclusão como abertura e não cria autorização de upload.
- **Plano de contingência:** manter participação indisponível até leitura válida, sem alterar datas persistidas.

### Dependência ainda não integrada

- **Probabilidade:** alta enquanto PP-STORY-003 estiver na feature.
- **Impacto:** promoção fora de ordem perder link/QR ou evidências.
- **Mitigação:** registrar base/SHAs, preservar a dependência na nova feature e promover PP-STORY-003 primeiro.
- **Plano de contingência:** reverter UI/API sem DDL; pendências físicas/acessibilidade seguem separadas.
