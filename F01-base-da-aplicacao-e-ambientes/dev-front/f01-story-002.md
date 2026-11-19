# F01-STORY-002 · Frontend local

A interface React e TypeScript apresenta a página inicial e o retorno para o início em rotas desconhecidas. O HTML declara `pt-BR`, a navegação oferece um link para pular ao conteúdo e os indicadores de foco ficam visíveis. [Interface](../../../scrapbook-ui/src/App.tsx) · [Rotas](../../../scrapbook-ui/src/routes/AppRoutes.tsx)

O Compose habilita o diagnóstico local no build. A pessoa usuária aciona a verificação, recebe o estado textual de `/health/ready` e pode repetir a solicitação quando o serviço estiver indisponível. O Nginx serve a SPA na mesma origem do proxy local com cabeçalhos de segurança e permissões de câmera e microfone desabilitadas. [Página inicial](../../../scrapbook-ui/src/pages/home/HomePage.tsx) · [Nginx](../../../scrapbook-ui/nginx/default.conf)

O `npm run check` passou com lint, build TypeScript/Vite e sete testes Vitest. O runner de navegador confirmou navegação por teclado, reflow e ausência de violações detectadas pelo axe nos cinco perfis executados. A matriz manual permanece pendente. [Evidência de validação](../test-funcional/2026-11-19_11-07-50/2-execucao.md)
