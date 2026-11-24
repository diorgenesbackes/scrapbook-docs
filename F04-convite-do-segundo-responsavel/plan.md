# Plano de implementação do convite do segundo responsável

## Pré-requisitos

- Usar a base F01 e a F03 já integradas em `dev`, com a F05 disponível para reutilizar a origem pública e o componente de QR. Sincronizar os repositórios afetados antes de abrir a primeira branch da story.
- Preparar a stack Docker local com o schema Liquibase vigente. Usar contas sintéticas em contextos de navegador independentes e a configuração privada de Development. [Ambiente local](../../scrapbook-infrastructure/README.md)
- Trabalhar com o recorte de convite por QR. A integração com login e cadastro reais fica na F02. O pedido e o encaminhamento por e-mail ficam na F12. [Spec](spec.md)

### Fase 1 · Base do convite

**1. Contratos** - Definir os contratos de emissão, consulta corrente, prévia e aceite. Alinhar estados, feedback e requisitos de privacidade com as telas S13 e S14.

**2. Persistência** - Integrar à aplicação as estruturas de convites, vínculos e contas já presentes no Liquibase. Compartilhar a fronteira transacional com as alterações de evento da F03.

**3. Proteção do link** - Preparar a geração, recuperação autorizada e resolução do token. Documentar a configuração local e os requisitos das chaves de cada ambiente.

### Fase 2 · Emissão e compartilhamento

**4. Emissão** - Entregar geração, consulta e recuperação de um convite pendente. Permitir substituição explícita e respeitar validade, disponibilidade de vaga e trava estrutural.

**5. Tela do responsável** - Integrar S13 ao detalhe do evento. Oferecer QR, link copiável, validade e mensagens para lotação, trava e falhas.

**6. Entrada do convite** - Publicar a rota de S14 e aplicar a política de privacidade do endereço. Garantir origem canônica para o QR nos ambientes configurados e origem loopback no Docker.

### Fase 3 · Aceite, integração e entrega

**7. Prévia para a conta convidada** - Identificar o evento no contexto de uma conta confirmada. Representar estados expirado, substituído, usado, lotado, travado e elegível.

**8. Aceite e auditoria** - Adicionar o segundo vínculo numa alteração atômica que consome o convite e registra a operação. Recuperar o resultado de uma resposta perdida e preservar as permissões equivalentes da F03.

**9. Tela de aceite** - Exigir confirmação explícita na S14 e encaminhar a conta ao detalhe depois do sucesso. Ligar o fluxo aos dados e à navegação existentes de “Meus eventos”.

**10. Integrações futuras** - Registrar os contratos de retomada após autenticação, resolução para e-mail e invalidação por trava. Vincular as responsabilidades às entregas da F02, F12 e F09.

**11. Entrega do recorte local** - Revisar o candidato conforme o ciclo de story do repositório. Registrar a jornada com contas sintéticas, o resultado da publicação e as integrações que ainda aguardam as respectivas features.
