# F10-STORY-001 · Diagnóstico das rodadas locais

O perfil é o mesmo nas rodadas com tráfego. Foram 80 convidados distribuídos igualmente em dois eventos, com 35 depoimentos de uma foto e cinco momentos de duas fotos por evento. A massa soma 90 JPEGs sintéticos e 185.030.650 bytes. A F08 executou consentimento, admissão, autorização, PUT em S3 versionado, conclusão, confirmação e consulta do recibo. A F10 consumiu o fato confirmado em SNS/SQS e preparou as variantes em duas réplicas com dois consumidores de foto por réplica.

O preparo no perfil Colima anterior esgotou o disco ao aplicar o bootstrap do PostgreSQL. A carga passou para o perfil descartável `scrapbook-f10-validation` com 4 vCPU, 6 GiB de RAM e 64 GiB de disco. O armazenamento do perfil anterior foi preservado. Na nova VM, a primeira configuração de notificação nativa do Moto não concluiu a confirmação do upload no aquecimento. A fixture passou a observar versão e tamanho de cada PUT real e entregar a evidência pelo SQS local, com timestamp de observação de seis dígitos, como no ensaio da F08.

## Primeira rajada

Os 80 VUs iniciaram e os 90 PUTs foram bem-sucedidos. O kernel encerrou o `moto_server` por falta de memória, com 5.063.084 KiB de RSS anônimo. O k6 registrou 58 respostas 504 na consulta de status e 22 recibos. O PostgreSQL conservou 22 fotos confirmadas, das quais 18 prontas. A coorte ficou incompleta, e a rodada foi encerrada com a remoção dos recursos descartáveis. [Evidência do ambiente](round-1/environment-failure.txt) · [Leitura final do PostgreSQL](round-1/partial-readiness.json)

A otimização global de cópia da fixture já versionada encerrou o aquecimento sem comprovar as duas variantes da F10. O ajuste temporário seguinte conservou as cópias normais e usou a cópia de metadados somente durante `list_object_versions`, com isolamento de contexto entre requisições. O aquecimento desse ambiente confirmou o recibo, ambas as variantes JPEG, bytes, dimensões e SHA-256.

## Rajada com relógio civil da VM

A coorte completa recebeu 80 recibos, sem erros HTTP. O banco publicou 84 pares até 300 s, de 90 fotos confirmadas (93,33%). A sonda da VM mediu degraus do relógio civil de −30,319 s e +30,321 s em relação ao monotônico. O journal registra `Time jumped backwards` e sincronizações do Lima com desvio de aproximadamente −30,319 s. O PostgreSQL rejeitou seis transições para `succeeded` na constraint `ck_processing_attempts_state`; `finished_at` ficava de 29,819 a 30,209 s antes de `started_at`. A observação monotônica confirmou as seis pendências. O tamanho dos JPEGs e a aplicação permaneceram iguais. [Sonda do relógio](round-clock-unstable/vm-clock-before.json) · [Rejeições sanitizadas](round-clock-unstable/constraint-details.json) · [Resultado do banco](round-clock-unstable/readiness.json)

Parar temporariamente o `systemd-timesyncd` apenas na VM descartável ainda produziu um degrau de +30,320 s. O relógio do host permaneceu com a configuração original. [Sonda com NTP parado](clock-stabilization/vm-clock-after.json)
