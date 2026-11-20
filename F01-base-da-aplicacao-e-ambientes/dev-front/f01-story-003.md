# F01-STORY-003 · Artefato da UI em homologação

O workflow gera o build estático com `VITE_API_BASE_URL` apontando para a origem HTTPS da API. A configuração de diagnósticos locais fica desabilitada. O pacote inclui `customHttp.yml` com as proteções de navegação e `release.json` com a revisão e a versão reservada.

O deploy manual do Amplify recebe o ZIP pela URL privada da própria execução. O workflow confirma a versão publicada pela origem HTTPS configurada e guarda o pacote e a evidência. A regra de fallback de SPA preserva `release.json` e outros arquivos estáticos. [Preparação da homologação](../dev-infra/f01-story-003-homologacao.md)
