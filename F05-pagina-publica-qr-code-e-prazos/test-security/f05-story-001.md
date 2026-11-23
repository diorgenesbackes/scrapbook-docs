# Verificação defensiva da consulta pública

No candidato integrado em `dev`, a análise estática verificou consulta parametrizada, projeção reduzida e uso explícito de credenciais omitidas no frontend. A suíte local comparou o retorno anônimo ao retorno com uma conta alheia e contribuições sintéticas no banco. As respostas apresentaram somente os campos públicos, sem identificador interno, configurações de abertura ou dados de recebimento.

Foram verificados `404` uniforme, `no-store`, correlação do Problem Details, `X-Robots-Tag`, `Referrer-Policy` e preservação de `X-Frame-Options`. Os logs locais de API e Nginx foram inspecionados pela suíte com busca de valores da fixture e da rota nominal. As amostras deram suporte à revisão sem registrar valores de eventos nas evidências. [Revisão técnica](../code-review/f05-story-001.md)

O artefato de publicação contém os headers esperados. O aceite de homologação deve verificar os headers da resposta publicada e conferir a coleta do edge e o perfil de limitação da F01. [Guia de observabilidade](../../guidelines/observabilidade.md)
