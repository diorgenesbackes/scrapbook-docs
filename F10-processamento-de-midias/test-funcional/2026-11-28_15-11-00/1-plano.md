# Plano de validação da F10-STORY-005

O candidato local usa os merges em `dev` do serviço `d7ac67d`, dos testes `c1782eb`, da infraestrutura `0ec49e6` e da documentação `bd724ec`. As dependências são UI `57f4e63` e Liquibase `ac0e851`. Os ensaios usam massa sintética e laboratório descartável. O aceite de homologação depende de candidato publicado, versões verificadas da API e do Liquibase e acesso autorizado à AWS. [Especificação](../../spec.md) · [PRD](../../../PRD.md) · [Cartão](https://trello.com/c/VIdGX9ti)

| Critério do cartão | Ação | Evidência exigida |
| --- | --- | --- |
| CA1 · Prontidão em 300 s | Observar todos os arquivos confirmados, da confirmação F08 ao commit de disponibilidade. | Denominador de 170 arquivos, pelo menos 162 em até 300 s, falhas e pendências incluídas, percentis por tipo e no total. |
| CA2 · Carga e recibos | Executar o perfil de dois eventos e a rodada de borda e observar o recibo. | 200 envios, 90 fotos, 40 áudios, 40 vídeos, pico efetivo, duração e resposta F08. Prova de formatos, tamanhos e durações de borda em homologação. |
| CA3 · Saídas e telemetria | Inspecionar variantes nos perfis e logs/métricas de processamento. | Originais e variantes privadas, payload dos sinais de cardinalidade limitada e os perfis processados no MediaConvert real. |
| CA4 · Ativação e recuperação | Pausar, confirmar novo envio, retomar e conferir efeitos do processamento. | Recibo, Outbox e original preservados, variantes prontas intactas, trabalho novo só após retomar. Em homologação, verificar também ativação gradual, drenagem, capacidade histórica e rollback no ambiente. |

Reúna artefatos agregados com o SHA de cada candidato. A fonte de commit e a continuidade temporal determinam se a observação pode ser usada no critério de 300 s. [Procedimento operacional](../../dev-infra/f10-story-005.md)
