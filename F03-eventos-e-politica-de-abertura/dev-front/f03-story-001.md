# F03-STORY-001 · Lista, formulário e detalhe

`/meus-eventos` mostra os eventos autorizados com paginação, carregamento, vazio, falha e sessão expirada. `/meus-eventos/novo` usa os horários civis de Brasília e sugere a data de abertura em um ano. A capa padrão reaproveita a imagem de celebração do design system. O detalhe mostra boas-vindas, os três marcos de envio e a data de abertura.

Quando o resultado do envio fica incerto, o formulário oferece recuperar a tentativa com a mesma chave e a mesma entrada. A interface conserva esses valores em memória até confirmar o resultado. O navegador usa dados de autenticação apenas no laboratório de testes, com credenciais sintéticas injetadas pelo Playwright. A integração de sessão real pertence à F02. [Meus eventos](../../design-system/screens/S09.md) · [Criar evento](../../design-system/screens/S10.md)
