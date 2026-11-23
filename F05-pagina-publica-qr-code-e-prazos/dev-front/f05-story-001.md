# F05-STORY-001 · Página dos convidados

`/e/{code}` abre diretamente no navegador. A página usa a capa padrão existente, nome, data em horário de Brasília e boas-vindas renderizadas como texto. O shell mantém o atalho de teclado para o conteúdo e a marca. Os módulos dos responsáveis são carregados sob demanda. [S16](../../design-system/screens/S16.md)

Antes da janela, a página informa a abertura. Durante a janela, exibe os dois botões desabilitados com “A participação estará disponível em breve.” Ao fechar, remove os botões. O hook calcula o tempo decorrido com relógio monotônico, reconsulta nas bordas com até dois segundos de atraso aleatório, a cada cinco minutos e ao voltar à aba. Ao trocar o código, cancela a consulta anterior. Código inválido e falha temporária têm recuperação própria. [Fluxo e prazos](../spec.md#fluxo-da-interface)

O GET público omite credenciais. O Nginx aplica `X-Robots-Tag` pela URI original e preserva os headers da F01 após o fallback da SPA. O pacote Amplify inclui as regras de headers para `/e` e `/e/*`. O aceite no ambiente publicado deve conferir o acesso direto e os headers efetivos. [Artefato de hospedagem](../../../scrapbook-ui/.github/scripts/package_amplify.py)
