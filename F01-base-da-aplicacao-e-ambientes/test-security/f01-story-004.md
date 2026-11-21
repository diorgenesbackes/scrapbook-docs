# F01-STORY-004 · Controles defensivos da recuperação

Os três workflows manuais passaram pela conferência de ref `hlg`, ambiente, permissões mínimas do job, Actions fixadas por SHA completo e passagem de inputs por variáveis de ambiente. A execução simulada recusou publicação de origem malsucedida, evidência de outro repositório, geração divergente, digest diferente, ZIP alterado e mudanças na definição ECS e no job Amplify. Os casos conferiram a ausência da chamada de ativação após a recusa.

O workflow do Liquibase recusou em simulação a instância sem a etiqueta de ensaio e a URL da tarefa divergente. No PostgreSQL descartável, a imagem recusou TLS sem verificação, hostname incompatível, banco diferente e reversão com dados. A ligação à AWS por OIDC, as permissões efetivas de IAM e a exposição no ambiente publicado continuam pendentes. [Ensaio em PostgreSQL](../test-funcional/f01-story-004-recuperacao-local.md)
