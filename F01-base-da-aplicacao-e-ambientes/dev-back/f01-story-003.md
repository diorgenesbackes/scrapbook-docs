# F01-STORY-003 · Configuração da API em homologação

A API usa `Hosting:PublicUiOrigin` para CORS com uma origem HTTPS explícita. A fundação aceita GET e HEAD e expõe a correlação ao navegador. A configuração local continua operando pelo proxy de mesma origem quando a variável está ausente.

O task definition define `Database:RequireTls`, com uma conexão `VerifyFull` e o bundle regional do RDS fixado na imagem. A identidade de banco permanece `scrapbook_app`. O Docker usa o SDK da plataforma de build e publica os assemblies portáveis na imagem de runtime da arquitetura alvo. A versão reservada compõe a versão informacional da imagem publicada e dos logs estruturados. [Preparação da homologação](../dev-infra/f01-story-003-homologacao.md)
