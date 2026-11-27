# Aceite funcional local da F10-STORY-001

A validação funcional aprovou CA1 a CA4 sobre o serviço integrado em `dev` no SHA `e9b6fe481d2b1cbd2a2e85fbe2ba6e60aa8dee9d`, em 27/11/2026. [Cartão da story](https://trello.com/c/3tAcFFKS) · [Resultados, comandos e SHAs do laboratório](../test-e2e/f10-story-001-dev.md)

| Critério | Resultado observado |
| --- | --- |
| CA1 | Cada foto válida do envio confirmado gerou um trabalho. O recibo chegou antes do preparo. Texto e conjunto rejeitado geraram zero trabalhos. As fotos recebidas antes do fechamento convergiram após o prazo. |
| CA2 | JPEG com orientação/GPS, PNG transparente, HEIC e HEIF genérico produziram as duas dimensões previstas, com proporção e orientação preservadas e localização removida. |
| CA3 | As duas referências privadas e o estado `available` entraram juntos. O ensaio também reteve a promoção de um arquivo para observar duas fotos do mesmo conjunto em estados diferentes. |
| CA4 | Duas réplicas retomaram após interrupção e reentrega e publicaram uma única dupla. Uma falha definitiva produziu o fato correlacionado e preservou recibo e versão original. |

O aceite humano em homologação deve repetir o fluxo com permissões e transporte do ambiente e conferir as variantes reais. [Aceite da F10](../../PRD.md#f10-processamento-de-mídias-2)
