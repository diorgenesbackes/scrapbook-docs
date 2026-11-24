# Revisão técnica da F06-STORY-001

Em 24/11/2026, a revisão abrangeu o formulário da S17, a rota local, a composição com a F05 e a suíte descartável. O [contrato da história](../dev-front/f06-story-001.md) define as fronteiras e dependências.

- O reconhecimento do código usa a forma já adotada pela F05. A consulta pública continua anônima e o formulário é desmontado na troca de evento. O modo de diagnóstico é explícito e restrito a loopback.
- Nome vazio e ausência de concordância bloqueiam a confirmação. O aviso é ligado à caixa, a primeira falha recebe foco e a correção do nome exige nova concordância.
- A confirmação entrega nome, código público, versão do aviso e instante ao consumidor local. A rota de prévia usa apenas o retorno genérico de sucesso. Nenhum campo sensível tem nome de serialização HTTP. O build normal foi inspecionado e não contém a rota de prévia nem o aviso sintético no JavaScript.
- A primeira rodada do teste por teclado passou em 13 de 15 casos. O Tab comum pulou controles em WebKit. A suíte passou a usar Option+Tab no WebKit, conforme a navegação nativa, mantendo as asserções de foco. O reteste passou em 15 de 15. [Atalhos do Safari](https://support.apple.com/guide/safari/keyboard-shortcuts-and-gestures-cpsh003/mac)

| Verificação | Resultado |
| --- | --- |
| `npm ci --offline --no-audit --logs-max=0` | Exit 0. Aviso `install-scripts` para `fsevents@2.3.3`, originado na instalação das dependências. |
| `npm run lint` | Exit 0, sem diagnósticos do Oxlint. |
| `npm run build` | Exit 0, sem avisos de TypeScript e Vite. |
| `npm test` | 55 testes passaram em 14 arquivos, sem coleta de cobertura numérica. |
| `dotnet restore ScrapBook.sln --locked-mode` e `dotnet build ScrapBook.sln --no-restore` | Exit 0, com zero erros e avisos na compilação. O serviço mantém o código da F05. |
| `git diff --check` | Exit 0 no escopo revisado. |

Sem achados técnicos remanescentes no escopo. O aviso oficial e a validação da autoria no envio permanecem nas respectivas integrações de produto.
