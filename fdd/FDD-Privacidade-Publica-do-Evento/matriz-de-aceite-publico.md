# Matriz de aceite da experiência pública

[FDD PP-006](FDD-Privacidade-Publica-do-Evento.md) · [História](https://trello.com/c/IV6uDrPq)

Cada linha referencia os critérios da história no quadro. Evidência histórica não é uma nova execução nem implantação. A revisão corrente foi concluída no [ciclo PP-006](../../testing/2026-11-13_21-42-37/2-execucao.md): 57 cenários browser, testes HTTP/SQL/sinais e unitários aprovados, com os limites abaixo.

| História / critérios | Evidência anterior preservada | Revisão PP-006 |
| --- | --- | --- |
| PP-001 AC01–05: acesso sem login, link direto/QR, código não é PK, shell guest e erros seguros | [Acesso](../FDD-Acesso-Publico-do-Evento/relatorio.md) | Comparar visitantes/atores e contrato exato; código não autoriza GET/PUT/share privados. |
| PP-002 AC01–05: apresentação, capa pública, texto seguro, responsividade e atualização | [Apresentação](../FDD-Apresentacao-Publica-do-Evento/relatorio.md) | SQL de quatro colunas, capa local, projeção permitida e navegação sem dados anteriores; regressão de apresentação aprovada. Leitor real PP-002-029 continua pendente. |
| PP-003 AC01–05: URL/QR/exportação coerentes, autorização e falhas sem sucesso falso | [Compartilhamento](../FDD-Compartilhamento-do-Evento/relatorio.md) | QR/PNG/print digitais inspecionados no runner de privacidade; conteúdo permitido e filename público. Papel/aparelhos PP-003-034 continuam pendentes. |
| PP-004 AC01–05: disponibilidade do servidor, bordas, tolerância e revalidação | [Janela](../FDD-Disponibilidade-dos-Envios/relatorio.md) | Mesmo payload com/sem registros, nos três estados de janela e dois modos de galeria. Tempo do probe é exclusivo do projeto de testes. |
| PP-005 AC01–05: link inválido, recuperação segura, estados distintos e falhas isoladas | [Recuperação](../FDD-Recuperacao-da-Pagina-Publica/relatorio.md) | Mesma dependência incorporada; redirect recusado, Referer ausente e diagnóstico edge seguro. Regressão de erros aprovada. |
| PP-006 AC01: dados públicos mínimos | [Plano](../../testing/2026-11-13_21-42-37/1-plano.md) | Lista exata, objetos aninhados, query efetiva e fixtures com dados privados. |
| PP-006 AC02: cliente sem prefetch/conteúdo privado | Mesmo ciclo | Rede/DOM/storage/cache/history, troca de evento, saída de gestão e cookies omitidos. |
| PP-006 AC03: contrato uniforme | Mesmo ciclo | Visitante, responsável e terceiro sintéticos; immediate/scheduled e antes/depois de registros. Cognito real posterior. |
| PP-006 AC04: código não concede autorização privada | Mesmo ciclo | GET/PUT/access/share existentes, com PK e código usados indevidamente. Galeria/mídia não implementadas, sem inferir proteção de futuras rotas. |
| PP-006 AC05: sinais e exportações | Mesmo ciclo | API/edge reais, spans próprios/MeterListener, QR/PNG/print digitais e redação de métodos arbitrários. Sem exportador externo. |

Manter aceite técnico local, aceite humano e deploy separados. A revisão legal/operacional de privacidade, retenção, menores e exclusão permanece requisito de lançamento do PRD, fora desta implementação de consulta pública.
