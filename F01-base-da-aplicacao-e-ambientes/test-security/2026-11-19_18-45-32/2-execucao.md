# Avaliação de segurança local da F01-STORY-002

**Resultado em 19/11/2026.** O achado P2 de escuta da identidade de desenvolvimento segue aberto. Os controles exercitados no Compose padrão passaram. [Plano e escopo](1-plano.md)

| Componente | Revisão em `dev` |
| --- | --- |
| API | `550872b` |
| UI | `024c436` |
| Infraestrutura | `bdf7347` |
| Testes | `9c9a15c` |
| Liquibase | `405538d` |
| Documentação | `1a8e778` |

## Achado confirmado

**F01-SEG-A01 · P2 · A validação da identidade pode classificar a escuta de forma diferente do Kestrel.** A validação reimplementa a seleção entre endereços do host e endpoints do Kestrel. A interpretação da preferência do host e a contagem de endereços após expansão divergem do runtime. Em configurações mistas, a validação pode considerar somente loopback quando o host seleciona escuta ampla. Isso compromete o opt-in exigido para a identidade sintética em `Development`. O achado foi confirmado estaticamente e permanece condicionado à configuração de escuta. O Compose testado publicou somente em loopback, e a recusa fora de `Development` passou.

Alinhar a validação aos endereços efetivos do runtime e retestar a interpretação e precedência de fontes de configuração. [Opções da API](../../../../scrapbook-service/ScrapBook.Api/Options/DevelopmentIdentityOptions.cs) · [Guideline de contêineres](../../../guidelines/backend.md#containers-locais) · [Evidência da revisão](../../code-review/f01-story-002.md#segunda-revisão-técnica)

## Evidência por perspectiva

| Controle | Perspectiva e identidade | Evidência | Resultado |
| --- | --- | --- | --- |
| SEG-01 · Configuração e origem de segredos | White box | A API consome configuração de execução. A UI recebe opções públicas de build. Os Dockerfiles usam imagens fixadas por digest e o Compose monta volumes separados para PostgreSQL e chaves da API. O Gitleaks 8.30.1 examinou dois commits por repositório em API, UI, infraestrutura e testes, no intervalo `origin/dev..dev`, com redação das evidências. Código `0` nos quatro, sem detecções. | Aprovado no recorte examinado |
| SEG-01 · Ordem de seleção de endereços | White box | Divergência demonstrada entre a configuração validada e a semântica do ASP.NET Core fixado. | P2 em aberto |
| SEG-02 · Identidade e erros | Grey box, identidades sintéticas | Passaram 25 unitários filtrados de opções, autenticação e handler, incluindo default desabilitado, token sintético válido e inválido e respostas sanitizadas. A tentativa de startup em `Production` com identidade habilitada encerrou com diagnóstico da opção. O token sintético foi exercitado no handler, pois as rotas operacionais são anônimas. | Aprovado nos cenários executados |
| SEG-03 · PostgreSQL | Grey box, `scrapbook_app` | A conexão com a role da aplicação confirmou 13 condições efetivas. A role tem SELECT, INSERT e UPDATE em contas, carece de CREATE no schema e de associação à proprietária, e carece de leitura das duas tabelas de controle do Liquibase. Em auditoria, INSERT está autorizado e SELECT, UPDATE, DELETE e TRUNCATE estão ausentes. | Aprovado nas permissões avaliadas |
| SEG-04 · Superfície HTTP | Black box guiado, anônimo | Sete consultas HTTP cobriram início, fallback, asset ausente, rota de API ausente e três endpoints de saúde. Confirmaram CSP, DENY, `nosniff`, `no-referrer` e restrições de câmera e microfone no HTML. A saúde respondeu JSON sanitizado e `no-store`; a API retornou UUID de correlação. Nenhuma das respostas HTML e de saúde examinadas emitiu cookie. A origem sintética nos endpoints de saúde ficou sem autorização CORS. | Aprovado nas respostas avaliadas |
| SEG-05 · Isolamento e logs | Grey box e black box guiado | As três publicações usaram `127.0.0.1`. API e UI rodaram sem root, com filesystem somente leitura, capabilities removidas e `no-new-privileges`. Conexão inválida encerrou o startup sem incluir a amostra sintética no diagnóstico. O marcador das duas consultas com query ficou ausente dos logs de API e proxy. | Aprovado nos controles avaliados |

O teste dinâmico usou um projeto Compose descartável com dados sintéticos e runtime compilado dos commits acima. Os recursos do projeto foram encerrados ao final. O black box foi guiado pelo inventário de rotas da entrega. Autorização de eventos, sessão de produto, mídia, AWS e ambientes implantados estão fora deste recorte. [Compose](../../../../scrapbook-infrastructure/compose.yaml) · [Runner utilizado para provisionar o alvo](../../../../scrapbook-tests/docker_stack.py)

## Dependências e artefatos

| Verificação | Resultado |
| --- | --- |
| `dotnet package list --vulnerable --include-transitive --no-restore --format json` | Código `0` nos cinco projetos, com fonte pública do NuGet. Nenhum pacote vulnerável retornado. |
| `npm audit --json --ignore-scripts` no frontend e nos testes | Código `0` em ambos. O npm listou 158 dependências no frontend e 6 nos testes, com zero avisos de vulnerabilidade. |
| `gitleaks git . --log-opts='origin/dev..dev' --redact=100` | Código `0` nas quatro execuções. A inspeção cobre os commits da história nesse intervalo. |
| Docker Scout, CVEs altos e críticos nas imagens de runtime | Bloqueado pela revisão automática antes da execução. A revisão identificou risco de envio de metadados de imagens e pacotes locais ao serviço externo. A cobertura de CVEs das camadas do sistema operacional permanece pendente. |

Os resultados de advisories descrevem as fontes consultadas nesta execução. A verificação dinâmica limitou-se a controles autorizados da stack local. As evidências brutas ficaram em diretório temporário privado.
