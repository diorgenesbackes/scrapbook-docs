# F01-STORY-003 · Artefato da UI em homologação

O job de build usa somente permissão de leitura no GitHub e gera o ZIP estático com `VITE_API_BASE_URL` apontando para a origem HTTPS da API. A configuração de diagnósticos locais fica desabilitada. O pacote inclui `customHttp.yml` com as proteções de navegação e `release.json` com a revisão e a versão reservada.

O job de publicação baixa o ZIP pelo ID do artefato da execução, confere o SHA-256, a origem da API, o SHA do commit e a versão, e só então assume a role AWS. O Amplify recebe o ZIP pela URL privada da própria execução. O workflow confirma a versão publicada pela origem HTTPS configurada e guarda a evidência. A regra de fallback de SPA preserva `release.json` e outros arquivos estáticos. [Preparação da homologação](../dev-infra/f01-story-003-homologacao.md)
