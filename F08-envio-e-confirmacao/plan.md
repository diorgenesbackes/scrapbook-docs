# Plano de implementação da F08 — Envio e confirmação

Complexidade **complexa** · 5 fases · 18 passos. O primeiro fluxo entregável é o depoimento em texto validado localmente. [Especificação](spec.md)

## Pré-requisitos

- F01 fornece API, UI, banco, migrations e stack descartável. F05 fornece código e janela. F06 já oferece identificação local e F07 fornece a composição revisada das duas categorias. [PRD e dependências](../PRD.md#8-grafo-de-dependências)
- Usar SDK .NET 10.0.401, Node 26.13.0 e npm 11.20.0, a versão de PostgreSQL prevista no ambiente e as migrations Liquibase aplicadas. Trabalhar com fixtures de tempo, evento, aviso e chaves técnicas próprias do laboratório. [Stack e guidelines](../guidelines/README.md)
- Ativação pública exige aviso aprovado com versão cadastrada e configuração do ambiente. As fases de mídia usam S3 privado versionado, credenciais técnicas e fila de evidências nas integrações. A homologação requer ambientes e perfis operacionais verificados. [Arquitetura e decisões](spec.md#2-impacto-na-arquitetura)

## Fase 1 — Preparar os contratos e a concordância

**1. Base local** — Comprovar as dependências F05–F07 e os esquemas já aplicados na stack descartável. Fixar os candidatos e a massa sintética usada para aceite.

**2. Aviso e concordância** — Integrar o aviso vigente à identificação do convidado. Entregar prova verificável de concordância associada ao evento e aos dados escolhidos na F06.

**3. Tentativa lógica** — Firmar os contratos públicos de admissão, status e recuperação. Exercitar a separação de acesso entre eventos e as decisões de prazo antes de abrir o envio.

## Fase 2 — Entregar envio de texto local

**4. Admissão de texto** — Admitir texto revisado com identificação válida pela janela do servidor. Aplicar a mesma regra temporal em toda a tentativa e verificar limites e consistência.

**5. Confirmação e primeiro fato** — Persistir o depoimento e concluir sua operação atomicamente. Disponibilizar o primeiro fato de confirmação à F09 pelo contrato da Outbox.

**6. Interface de envio e recibo** — Ligar a revisão F07 às etapas S26 e S27. Entregar recuperação de resposta perdida e novo ingresso conforme a janela aberta.

**7. Entrega local de texto** — Validar a jornada completa no Docker com PostgreSQL e aviso sintético. Fechar as evidências de concorrência, privacidade, prazos e confirmação antes de acrescentar o transporte de mídia.

## Fase 3 — Acrescentar recepção de mídia

**8. Manifesto e autorizações** — Estender a tentativa ao conjunto ordenado de arquivos. Preparar o armazenamento privado versionado e a emissão de autorizações por arquivo.

**9. Recepção de fotos** — Integrar transferência, progresso, retentativa e coleta de evidência de recepção para fotos. Exercitar a chegada de um conjunto completo à etapa de validação.

**10. Vídeo e áudio** — Integrar a transferência dos formatos restantes. Usar multipart para vídeos grandes e retomar apenas partes elegíveis da mesma sessão.

**11. Prazos e reconciliação** — Comprovar o recebimento integral com o armazenamento e reconciliar perda de resposta, interrupção e notificações atrasadas. Preservar os marcos admitidos ao retomar o trabalho.

## Fase 4 — Confirmar conjuntos e recuperar falhas

**12. Validação dos bytes** — Validar tamanho, integridade, formato real e duração ou imagem dentro do perfil operacional. Preservar a origem rastreável para a confirmação.

**13. Preservação e confirmação integral** — Preservar versões privadas e confirmar o conjunto como unidade. Registrar os fatos transacionais para trava e processamento conforme os contratos da F09 e F10.

**14. Saneamento e limpeza** — Retomar operações externas de resultado incerto e limpar temporários elegíveis. Verificar que original confirmado e proteção de idempotência seguem resguardados.

**15. Contratos posteriores e operação** — Entregar a projeção de registro confirmado à F11 e os sinais sanitizados de falha à F13. Definir o perfil de trabalho e registrar as fronteiras que serão verificadas com cada consumidor.

## Fase 5 — Validar o candidato e preparar homologação

**16. Aceite integrado** — Validar jornadas de texto e mídia, estados de erro, precisão dos marcos, replays e consistência sob concorrência. Exercitar segurança defensiva, acessibilidade e a matriz de compatibilidade indicada na especificação.

**17. Carga e recuperação** — Medir o perfil de 200 envios em dois eventos e o p95 das operações de API. Demonstrar retomada, atraso de limpeza e recuperação entre efeitos de armazenamento e banco.

**18. Evidências e promoção** — Registrar candidatos, dependências, resultados e pendências por repositório. Aplicar o fluxo de story para `dev` e `hlg` e entregar a matriz do aceite de homologação, distinguindo dados sintéticos de integrações reais.
