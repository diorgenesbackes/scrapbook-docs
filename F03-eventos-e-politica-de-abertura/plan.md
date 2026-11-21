# Plano de implementação de eventos e política de abertura

## Pré-requisitos

- Iniciar as branches a partir de `dev` atualizado. Identificar as dependências entre serviço, UI, Liquibase, documentação e validação ao criar as histórias.
- Ter o schema `scrapbook` da F01 aplicado em PostgreSQL local e a fundação de API, UI e execução local disponível. Usar .NET SDK 10.0.401, Node 26.13.0, npm 11.20.0, PostgreSQL 18.6 e Liquibase 4.33.0, conforme a [base da F01](../F01-base-da-aplicacao-e-ambientes/spec.md).
- Habilitar a identidade de Development somente no laboratório e preparar duas contas sintéticas para provar isolamento. A F02 fornecerá a identidade real na integração.
- Usar a capa padrão nesta entrega. As boas-vindas podem ser personalizadas após a primeira história. Consultar a [spec](spec.md) para contratos, transições e critérios de aceite.

### Fase 1. Criação e listagem locais

**1. Base executável e identidade local** - Preparar a execução integrada da fundação e as contas sintéticas do laboratório. Confirmar que cada identidade tem uma conta persistida e que a configuração de desenvolvimento fica restrita ao ambiente local.

**2. Persistência da criação** - Preparar a gravação conjunta do evento e do primeiro vínculo. Registrar a repetição segura do comando e verificar a evolução da base em PostgreSQL descartável.

**3. Fluxo de criar e listar** - Implementar a primeira história com nome, data e horário, capa e boas-vindas padrão e abertura futura com sugestão ajustável. A listagem mostra os eventos vinculados à conta autenticada.

**4. Detalhe inicial e recuperação de tentativa** - Abrir o detalhe após criar e confirmar que uma resposta perdida pode ser recuperada sem duplicar o evento. Exercitar o isolamento com as duas contas.

### Fase 2. Configuração e política de abertura

**5. Edição das informações** - Permitir alterações de nome, data, horário e boas-vindas enquanto a estrutura estiver editável. Atualizar os prazos associados ao horário e preservar dados diante de um conflito.

**6. Escolha da abertura** - Oferecer escolha entre abertura imediata e futura durante a criação e em configuração própria. Validar datas em Brasília e explicar como recuperar uma sugestão incompatível.

**7. Transições temporais e travas** - Aplicar o fechamento ao meio-dia, a transição única a partir do imediato e a preservação das travas em alterações de horário. Integrar a autorização da área fechada ao relógio do servidor.

**8. Registro das mudanças de política** - Persistir a alteração aceita e sua trilha juntas. Conferir conflitos entre duas sessões, recusa de transições inválidas e recuperação após falha.

### Fase 3. Contratos para integração

**9. Configuração dos responsáveis e página pública** - Disponibilizar os dados que F04 e F05 consomem. Conferir a igualdade dos dois responsáveis, o código estável e os prazos atualizados após alterações permitidas.

**10. Trava estrutural, galeria e avisos** - Entregar os contratos de integração de F09, F11, F12 e F13. Exercitar localmente a trava, a liberação por data, a política vigente e a trilha de auditoria.

**11. Operação da entrega local** - Consolidar a execução pelo ambiente de desenvolvimento e registrar as evidências técnicas e funcionais nas pastas da feature. Identificar as verificações que serão concluídas com a identidade real e as features consumidoras.
