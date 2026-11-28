# F10-STORY-004 · Recuperação histórica sob carga

Validação local independente de CA1 e T11 em 28/11/2026. O perfil é o de carga com recuperação histórica descrito no README de `scrapbook-tests`. [Cartão](https://trello.com/c/dLVALXU9)

## Candidato e ambiente

| Repositório | SHA integrado em dev |
| --- | --- |
| scrapbook-service | `3834529e86e1123a4c3119c67be5395478ea6612` |
| scrapbook-liquibase | `ac0e8517af47cdf4a61b81c715b0e8544040ecde` |
| scrapbook-infrastructure | `8f6dd8cfb4226c467c48fc63e34667a82fb4b17a` |
| scrapbook-tests | `4a720663ae2f1a398ba941bb2f7c10a6cc674822` |
| scrapbook-ui | `57f4e63f9a6378ecde532d53b4d204ac935021ff` |

Colima local `scrapbook-f10-validation`, Ubuntu 24.04.4 arm64, 4 vCPU e 6.197.444.608 bytes de RAM visíveis ao Docker. Docker Engine 29.5.2, cliente 29.8.2, Compose 5.6.0, PostgreSQL 18.6, Liquibase 4.33.0, AWS CLI 2.37.10 e Python 3.13.15. O SDK do build .NET estava fixado em 10.0.401. A máquina ficou reservada durante as rodadas, com base e storage novos. Moto emulou S3 versionado, SNS e SQS em loopback. Os comandos AWS CLI usaram endpoints locais e credenciais sintéticas.

Por evento, oito registros de cinco arquivos históricos e seis registros novos de um arquivo, com JPEG sintético 32 × 32 e 793 bytes por arquivo. Duas réplicas do serviço executaram um slot exclusivo de novos e um slot elegível para o histórico em cada réplica. O preparo, com admissão e PUT, precedeu a fase medida. Cada página CLI tinha até 40 arquivos. As notificações para os 12 arquivos novos foram liberadas com histórico já admitido, em dois lotes, um por evento. A janela por arquivo foi de 300 s. Os cinco unitários do runner passaram.

## Primeira rodada com o relógio civil da VM

O runner publicado confirmou 16 recibos históricos, admitiu 40 fotos por evento e confirmou os 12 recibos novos. A sonda reuniu 1.057 pontos a cada 250 ms e observou 44 passos de pelo menos 100 ms, entre −30,729 s e +30,727 s. O PostgreSQL recusou cinco tentativas de promover para `succeeded` em `ck_processing_attempts_state`. Nessas transições `finished_at` antecedia `started_at` entre 30,590 s e 30,621 s. Antes da interrupção controlada, 80 históricas e 11 novas estavam prontas e uma nova permanecia em execução. O relógio invalidou as latências dessa rodada. O runner encerrou e a limpeza confirmou zero contêineres, redes e volumes do projeto. [Diferenças sanitizadas](load-independent-dev/constraint-sanitized.json) · [Drenagem imediatamente anterior à interrupção](load-independent-dev/drain-sanitized.txt) · [Sonda agregada](load-independent-dev/clock.json)

## Reteste com relógio monotônico

A primeira calibração foi interrompida no preparo porque o desvio da âncora UTC do host produziu `X-Amz-Expires=631`, acima do limite de 600 s verificado pela fixture. O reteste calibrado usou o patamar inferior observado do relógio da VM, ligeiramente anterior ao relógio da assinatura. O TimeProvider da API e do CLI avançou pelo monotônico do host em uma cópia temporária do serviço. A fixture escreveu no mesmo inode e preservou a verificação da assinatura e o perfil de carga. O host e a VM mantiveram seus relógios de sistema.

A latência histórica começa no início da chamada da página CLI. A latência nova começa no `confirmedAt` emitido pela API com a fixture. A prontidão é a primeira observação da promoção de `image` e `thumbnail` no PostgreSQL, com polling nominal de 0,5 s. Os números são cotas superiores que incluem a espera pela observação. O histórico também inclui o tempo entre o início da página e a admissão de cada arquivo. A sobreposição exige uma leitura consistente em que o histórico já observado antes da rajada ainda esteja pendente e o envio novo já esteja confirmado ou pronto. A consulta usa estados do banco sem subtrair timestamps do relógio civil da VM.

| Coorte | Prontas em até 300 s | p50 (s) | p95 (s) | p99 e máximo (s) |
| --- | ---: | ---: | ---: | ---: |
| Histórico, total | 80/80 | 2,463 | 4,301 | 4,808 |
| Histórico, evento A | 40/40 | 2,463 | 4,301 | 4,808 |
| Histórico, evento B | 40/40 | 2,463 | 4,301 | 4,808 |
| Novas, total | 12/12 | 0,821 | 1,064 | 1,064 |
| Novas, evento A | 6/6 | 0,734 | 1,064 | 1,064 |
| Novas, evento B | 6/6 | 0,883 | 1,039 | 1,039 |

Nos percentis pelo posto mais próximo, o p95 das novas caiu na 12ª amostra ordenada e equivale ao máximo. O p95 das históricas caiu na 76ª de 80. Os dois marcadores de início são diferentes, portanto os percentis das duas coortes descrevem intervalos distintos. [Contagens e tempos por evento](load-independent-monotonic-aligned/profile/readiness.json)

As duas páginas examinaram e admitiram 40 arquivos cada, sem rejeições nem adiamentos. Na observação imediatamente anterior à rajada, as 80 históricas estavam admitidas e pendentes. A rajada de notificações durou 0,261 s. As 12 novas confirmaram e ficaram prontas com pelo menos um desses trabalhos históricos ainda pendente. A janela medida da abertura das páginas até a última observação durou 4,876 s, com média de 18,87 arquivos prontos por segundo nessa janela. Houve 16 recibos históricos e 12 novos, zero amostras ausentes, falhas ou pendências. A sonda do relógio civil registrou seis saltos entre −30,692 s e +30,680 s durante o reteste, com intervalo máximo de amostragem de 0,261 s. A execução terminou em 37,393 s, incluindo preparo e limpeza, com código zero. A verificação posterior encontrou zero contêineres, volumes e redes do projeto. [Contagens e tempos por evento](load-independent-monotonic-aligned/profile/readiness.json) · [Sonda agregada do reteste](load-independent-monotonic-aligned/clock.json)

CA1 e T11 atendidos para a concorrência e a prontidão neste perfil local de JPEG com relógio monotônico de fixture. O resultado de 12/12 atende à meta de 95% em 300 s para as mídias novas desse perfil. A carga mista de referência da F10, arquivos de borda, uso do MediaConvert real e capacidade do ambiente implantado seguem como ensaios separados na especificação.
