# Plano de implementação da página pública, QR code e prazos

## Pré-requisitos

- Iniciar as branches a partir de `dev` atualizado em service, UI, testes, infraestrutura e documentação. O Liquibase não muda nesta feature.
- Ter a F03 integrada em `dev`, com `events.code` gerado na criação e os três marcos da janela persistidos. Usar as versões da [base da F01](../F01-base-da-aplicacao-e-ambientes/spec.md) e as contas sintéticas da [F03](../F03-eventos-e-politica-de-abertura/spec.md) nas telas do responsável.
- Escolher e fixar a biblioteca de QR antes da Fase 2, registrando versão, origem, licença e avisos conforme os [princípios](../guidelines/principios.md#manutenção-de-dependências).
- Definir `VITE_PUBLIC_SITE_ORIGIN` em cada ambiente publicado. O domínio definitivo é necessário antes de imprimir o QR do piloto, mas não para o desenvolvimento local. Consultar a [spec](spec.md) para contratos, estados e critérios de aceite.

### Fase 1. Página pública e janela

**1. Consulta pública por código** - Implementar a leitura por código com os campos permitidos, `404` uniforme e `no-store`. Verificar que a rota funciona sem identidade e que os logs registram somente a rota nominal.

**2. Estado da janela** - Calcular `before`, `open`, `closed` e a próxima transição no servidor, com relógio controlado nas bordas e nas viradas de dia e de ano. Deixar a regra pronta para a admissão da F08.

**3. Página dos convidados** - Criar `/e/{code}` no shell de convidado, com capa, nome, data, boas-vindas e estados da janela. Tratar link inválido, indisponibilidade e reconsulta nas transições, na volta à visibilidade e no intervalo periódico.

**4. Privacidade da superfície pública** - Aplicar `X-Robots-Tag` em `/e/` sem perder os headers de segurança. Conferir resposta, HTML e telemetria sem dados privados.

### Fase 2. Link e QR para o responsável

**5. Link do evento** - Expor o código somente ao responsável vinculado e compor o link com a origem pública configurada.

**6. Tela de compartilhamento** - Abrir a tela pela configuração do evento, copiar o link com alternativa manual e anunciar o resultado.

**7. QR code** - Gerar a prévia, baixar PNG e SVG e imprimir com a folha própria. Reusar o componente na seção de compartilhamento da página dos convidados, carregado sob demanda.

### Fase 3. Integração e preparação para o piloto

**8. Integração com a F03** - Editar nome, boas-vindas e horário antes da trava e conferir a página pública, os marcos e a validade do link.

**9. Contratos para F06, F07 e F08** - Entregar o contexto público, os destinos de ingresso e a regra de janela para as próximas etapas de participação. Aplicar o estado provisório dos botões até a composição estar disponível.

**10. Preparação para distribuição** - Preparar o endereço definitivo, o material para impressão e a proteção de entrada por ambiente antes da distribuição do QR. Usar os critérios da especificação para o aceite.
