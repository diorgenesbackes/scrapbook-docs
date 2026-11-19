# Plano de implementação da base da aplicação e ambientes

## Pré-requisitos

- Usar os seis repositórios e o [Git Flow](../guidelines/git-flow.md). Vincular a entrega ao quadro Scrap Book e identificar as dependências entre repositórios antes de abrir os PRs.
- Disponibilizar .NET SDK 10.0.401, Node 26.13.0, npm 11.20.0, Docker com Compose, PostgreSQL 18.6 local e Liquibase 4.33.0. Fixar ferramentas, dependências e imagens conforme a [especificação](spec.md).
- Ter configuração privada local e acesso operacional ao GitHub e à AWS para preparar homologação e produção. Identificar os domínios, certificados e referências de segredos de cada ambiente antes da publicação.
- Conferir se o banco alvo está vazio para a instalação inicial. Se houver estrutura ou histórico de migração anterior, identificar sua origem e definir o procedimento de transição antes de aplicar a base.

### Fase 1. Estrutura da aplicação

**1. Contratos e responsabilidades** - Alinhar a responsabilidade de cada repositório, a base documental e a ordem de entrega dos componentes.

**2. Serviço da aplicação** - Criar a solução com os projetos da Clean Architecture, a composição HTTP e a fronteira de tratamento de falhas definidas na especificação.

**3. Base do frontend** - Criar a aplicação web, sua estrutura inicial de rotas e a superfície mínima responsiva e acessível.

**4. Configuração operacional** - Preparar os contratos de configuração, os sinais de saúde e prontidão e o diagnóstico mínimo para a operação.

### Fase 2. Banco e execução local

**5. Modelo físico inicial** - Materializar em uma única entrega o inventário de tabelas e restrições previsto para a base. Atualizar a referência vigente de modelagem e preservar a versão histórica.

**6. Instalação e reversão do banco** - Preparar o fluxo do Liquibase, a separação das identidades e os procedimentos de instalação, reaplicação e recuperação.

**7. Ambiente local reproduzível** - Empacotar os componentes e conectar aplicação, frontend, banco e migrador no ambiente local.

**8. Operação local** - Registrar como iniciar, verificar, recuperar e atualizar a base mantendo o estado persistente.

### Fase 3. Ambientes AWS

**9. Infraestrutura por ambiente** - Preparar homologação e produção com limites próprios de rede, computação, banco e armazenamento.

**10. Acesso e exposição** - Conectar os serviços nas origens autorizadas e preparar HTTPS, proteção de entrada e acesso operacional.

**11. Dados e identidades** - Preparar os recursos de banco, arquivos e segredos com separação entre ambientes e entre execução e migração.

**12. Observabilidade operacional** - Disponibilizar sinais de saúde, estado da implantação e diagnósticos suficientes para operar cada ambiente.

### Fase 4. Promoção e recuperação

**13. Fluxo de integração** - Configurar as verificações e permissões das branches conforme o Git Flow da entrega.

**14. Versões e artefatos** - Preparar a reserva de versões e os registros que vinculam código, artefato e execução de publicação.

**15. Publicação dos componentes** - Conectar os fluxos independentes de frontend, backend e banco aos destinos previstos. Registrar a ordem necessária quando houver dependências entre esses fluxos.

**16. Recuperação operacional** - Preparar os procedimentos de retorno dos componentes e a recuperação controlada do banco.

**17. Promoção da base** - Promover a entrega a homologação e depois a produção conforme as evidências de aceite aplicáveis. Registrar a referência da base que será usada por Contas e sessões.
