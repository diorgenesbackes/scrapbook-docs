# FDD: Compartilhamento por link e QR code

**História:** [PP-STORY-003](https://trello.com/c/yUy0Yq2f), versão 0.1, implementada e validada localmente em feature/PP-STORY-003; leitura física pendente, sem merge/deploy.

[PRD FR-005](../../PRD.md#fr-005-página-dos-convidados-e-qr-code) · [HLD](../../HLD.md#interfaces-públicas) · [S15](../../visual-identity/screens/S15.md) · [S16](../../visual-identity/screens/S16.md)

## 1. Contexto e objetivo

Compartilhar o mesmo acesso público por link, QR exibido, PNG e material impresso. O responsável autorizou parar a stack e integrar PP-STORY-002 em quatro mains antes desta feature. Árvores iguais às versões validadas; PP-002-029 de leitor real continua pendente mesmo após o merge. A nova US não será integrada automaticamente.

## 2. Escopo e exclusões

S15 autorizado aos dois responsáveis, acesso a partir do detalhe do evento, copiar/alternativa manual, PNG com metadados, composição de impressão e QR público em S16. Não criar convites, login, captura/envio, galeria, serviço de encurtamento, armazenamento de QR, DDL ou AWS. Link/QR identificam o evento; não autorizam gestão, mídia privada ou envio.

## 3. Origem canônica e configuração

PublicSite:Origin é configuração confiável de origem absoluta, sem path/query/fragment/userinfo. HTTPS externo obrigatório; HTTP permitido exclusivamente para loopback com PublicSite:AllowHttpLoopback=true. Valores inválidos recusam a inicialização. Valor ausente permite a apresentação, mas não inventa links. Normalizar esquema/host via Uri, preservar código case-sensitive. Nunca construir a URL por Host, X-Forwarded-Host/Proto, query ou window.location.

Compose local usa SCRAPBOOK_PUBLIC_ORIGIN, padrão http://127.0.0.1:5173, e SCRAPBOOK_PUBLIC_ALLOW_HTTP_LOOPBACK=true. Helpers up/run-api calculam o fallback pela UI_PORT quando a origem não foi definida. Ao usar Compose manual com outra porta, configurar a origem explicitamente. Não há segredo nessa variável. Development nativo tem padrão local explícito; Production sem configuração externa retorna link indisponível. Domínio externo real continua pendente de decisão Q04; exemplos .invalid são sintéticos, não defaults do produto.

## 4. Contratos e arquitetura

| Contrato | Definição |
| --- | --- |
| SPA responsável | /events/:eventId/share, retorno ao detalhe correspondente. |
| API privada | GET /api/v1/events/{eventId}/share, operationId GetEventSharing. |
| Resposta privada | code, name, publicUrl; sem informações de recebimentos ou permissões. |
| Apresentação pública | Acrescenta publicUrl obrigatório/nulável à resposta anterior; NULL quando não configurado. |
| URL | Origem normalizada + /e/ + código público existente; todos os formatos usam essa mesma string. |
| Erros | 401 sem identidade; 404 alheio/ausente; 400 PK/query inválida; 503 SHARING_NOT_CONFIGURED sem origem após verificar vínculo; demais erros existentes. |
| Headers/orçamento | no-store, correlação e 5s incluindo resolução de identidade; cancelamento até EF. |

EventSharingRepository faz uma consulta AsNoTracking com filtro de vínculo por accountId/eventId e projeção code/name. Sem carregar entidade inteira, consultar registros ou gravar auditoria. Slots 1 e 2 têm o mesmo acesso. EventSharingService monta a resposta somente após autorização. PublicEventService reutiliza PublicEventLinkPolicy para S16. Models, Responses e Policy são puros em Domain; Services/portas em Application; opções/HTTP/OpenAPI na API e EF em Infrastructure. Sem dependência de framework em Domain/Application.

A UI valida protocolo/origem/código/path do link recebido e não substitui sua origem. URL com credenciais/query/fragment, outro código ou HTTP externo não é aceita. Clientes antigos continuam usando campos anteriores; publicar API/configuração antes da nova UI. Sem mudança de banco, geração/rotação de código ou migração de QR persistido.

## 5. QR, download e impressão

Gerador [qrcode 1.5.4](https://github.com/soldair/node-qrcode), licença MIT, e tipos @types/qrcode 1.5.6 MIT, fixados em manifest/lockfile. QR preto sobre branco opaco, correção M, quiet zone de quatro módulos, bitmap de 1024px, sem logo sobreposto. A prévia usa PNG data URI local, sem serviço externo ou chamada de mídia. O link textual continua disponível quando a geração falhar.

PNG de download: canvas de 1280px de largura, nome em texto seguro acima, QR de 1024px ao centro e URL legível abaixo; altura ajustada às linhas. Quebra por grafemas para preservar Unicode. Nome de arquivo scrapbook-evento-{code}.png, sem nome de usuário no path. Blob deve ser PNG não vazio; URL temporária revogada após 1s ou desmontagem, anchor removido imediatamente. Async antigo não publica resultado em outra rota. Exportação tem trava independente da geração/cópia.

Impressão usa um portal no body com nome/link/QR, fundo branco e QR de 70mm; durante print, #root e controles ficam fora da composição. O próprio documento é impresso, sem pop-up. beforeprint/afterprint e cleanup removem o estado temporário, com limite de 30s. Se window.print faltar/falhar, orientar menu do navegador. Não é possível confirmar pelo JavaScript se papel saiu ou a janela foi silenciosamente bloqueada; feedback não afirma impressão concluída. Ctrl+P tem a composição quando o material está pronto.

## 6. Estados e acessibilidade

Carregamento autorizado, erro de consulta e retorno contextual. Geração de QR, cópia, exportação e impressão independentes. Copiar só confirma após Clipboard API resolver; falha disponibiliza input readonly focado/selecionado para cópia manual. Falha de geração permite retry sem retirar o link. PNG ausente/inválido não inicia download. Botões têm estado e nome visível, 48px, foco; output anuncia feedback sem cor como único sinal.

S16 oferece QR/link e copiar sem exigir login; download/impressão ficam em S15 para responsáveis. Desmontagem descarta callbacks e recursos temporários. Não prometer funcionamento de Clipboard/impressão em todos os navegadores; cobrir fallback. Sem novos controles de contribuição enquanto destinos não existirem.

## 7. Observabilidade e privacidade

events.share_get / events.sql.share_get, rota nominal e correlação, sem código, URL utilizável, nome ou PNG em logs/tags. Scopes automáticos do console continuam desativados. Dados privados não entram na API, DOM, metadados do arquivo ou QR. O código público não substitui PK, identidade, vínculo ou autorização de mídia. Snapshots das tabelas verificam somente leitura.

## 8. Validação

Unitários xUnit/Vitest de origem, códigos, autorização, cópia e fallback, falha/export vazio, concorrência e liberação. HTTP com dois responsáveis, outsider, PK/código inválido, Host adulterado, origem HTTPS externa sintética, origem ausente e acesso público em Production sem identidade. Testes manuais usam Compose/portas/volumes exclusivos e mesmas 47 migrations.

Leitor independente [jsQR 1.4.0](https://github.com/cozmo/jsQR), Apache-2.0, e [pngjs 7.0.0](https://github.com/pngjs/pngjs), MIT, apenas em scrapbook-tests. Decodificar bitmap exibido, screenshot da prévia, PNG exportado e screenshot de mídia print; comparar byte a byte com a URL canônica e abrir em contexto anônimo. Validar quiet zone digital. QR de origem externa é testado sem acessar o domínio sintético.

PP-003-034 exige aparelhos reais e papel: documentar modelo/câmera/leitor, tamanho impresso, distância/luz, URL decodificada e evento alcançado. Decodificação digital não substitui essa etapa. Se não executada, manter o item em Test/incompleto, sem declarar leitura física validada. A pendência de leitor de tela real anterior permanece separada.

## 9. Compatibilidade, recuperação e riscos

API/config antes da UI. Reverter UI e depois API/config, preservando dados. Sem DDL/DML de funcionalidade; imagens e recursos de testes são descartáveis, sem iniciar a stack persistente. Domínio externo, aparelho/papel e revisão humana não são inferidos de screenshots. Async gerador/clipboard não é abortável nativamente; callbacks são ignorados ao sair. URL existente pode continuar compartilhável; não introduzir expiração/rotação de código sem requisito.

[Relatório e evidências](relatorio.md) detalham revisões, resultados, falhas/retestes e transições observadas em UTC. São 35 subtarefas em To Deploy e PP-003-034 em Test/incompleta; a história permanece em Test. Não inferir horas ativas desses horários.

## 10. Subtarefas

| ID | Entrega |
| --- | --- |
| PP-003-001 | [docs] Definir uma única regra de URL canônica usada por compartilhamento, QR público, download e impressão. |
| PP-003-002 | [infrastructure] Definir configuração da origem pública por ambiente; não montar o link a partir de Host arbitrário do request. |
| PP-003-003 | [docs] Registrar a decisão pendente de domínio e permitir base local configurada sem publicar links fictícios. |
| PP-003-004 | [docs] Definir contrato mínimo de compartilhamento para a gestão, preservando compatibilidade com a resposta existente. |
| PP-003-005 | [docs] Escolher formato de download, dimensões, contraste e quiet zone com critério verificável de leitura. |
| PP-003-006 | [ui] Selecionar gerador QR compatível, registrar licença e fixar versão/lockfile sem versionar bibliotecas. |
| PP-003-007 | [service/Application] Reutilizar o código público do evento na consulta autorizada de compartilhamento. |
| PP-003-008 | [service/Infrastructure] Buscar dados mínimos de compartilhamento sob a verificação existente de vínculo com o evento. |
| PP-003-009 | [service/Api] Expor a composição escolhida do contrato de compartilhamento com erros de autorização coerentes. |
| PP-003-010 | [service/Api] Atualizar OpenAPI com acesso permitido, resposta e regras da URL pública. |
| PP-003-011 | [infrastructure] Validar a origem configurada e documentar HTTPS externo e exceção HTTP para desenvolvimento local. |
| PP-003-012 | [infrastructure] Propagar configuração pública necessária à UI/container sem incluir segredos. |
| PP-003-013 | [ui] Adicionar acesso ao compartilhamento a partir do detalhe do evento, preservando o retorno contextual. |
| PP-003-014 | [ui] Criar a tela S15 com nome do evento e link público legível. |
| PP-003-015 | [ui] Implementar copiar link com Clipboard API e feedback acessível de sucesso. |
| PP-003-016 | [ui] Implementar alternativa de seleção/cópia manual quando a área de transferência estiver indisponível. |
| PP-003-017 | [ui] Criar componente QR reutilizável que codifique exatamente a URL canônica. |
| PP-003-018 | [ui] Exibir QR e link textual alternativo na página pública S16. |
| PP-003-019 | [ui] Exibir QR e link na tela de compartilhamento S15. |
| PP-003-020 | [ui] Implementar download do formato escolhido com nome de arquivo seguro. |
| PP-003-021 | [ui] Liberar recursos temporários de exportação após download ou desmontagem. |
| PP-003-022 | [ui] Implementar composição de impressão branca com nome, link e QR, sem navegação ou botões. |
| PP-003-023 | [ui] Acionar impressão sem perder o contexto e tratar bloqueio ou indisponibilidade do navegador. |
| PP-003-024 | [ui] Implementar estados independentes de geração e exportação, evitando downloads duplicados durante processamento. |
| PP-003-025 | [ui] Anunciar erros e sucesso sem depender apenas de cor, animação ou imagem. |
| PP-003-026 | [service/UnitTests] Cobrir origem configurada, código existente e recusa de origem inválida. |
| PP-003-027 | [tests/HTTP] Validar compartilhamento para os dois responsáveis e negar ator sem vínculo. |
| PP-003-028 | [tests/HTTP] Validar que o código de compartilhamento não libera as rotas privadas. |
| PP-003-029 | [ui/test] Cobrir cópia com sucesso, permissão negada e alternativa manual. |
| PP-003-030 | [ui/test] Cobrir geração, exportação e erro sem disparar download vazio ou incorreto. |
| PP-003-031 | [tests/QR] Decodificar QR exibido e arquivo exportado usando leitor independente do gerador. |
| PP-003-032 | [tests/E2E] Abrir a URL decodificada em contexto anônimo e conferir o evento correto. |
| PP-003-033 | [tests/E2E] Conferir nome/link/QR na mídia de impressão e ausência de controles administrativos. |
| PP-003-034 | [tests/manual] Validar leitura em aparelhos reais e impressão em papel, registrando dispositivos e condições. |
| PP-003-035 | [tests/E2E] Conferir isolamento de origem entre configuração local e ambientes externos. |
| PP-003-036 | [docs] Documentar operação, formato, configuração de domínio, limitações dos navegadores e evidências. |
