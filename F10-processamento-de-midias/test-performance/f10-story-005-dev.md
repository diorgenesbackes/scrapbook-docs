# Prontidão da carga mista da F10-STORY-005

O perfil usa 200 convidados distribuídos igualmente entre dois eventos sintéticos. São 50 textos, 70 envios de foto com 90 arquivos, 40 envios de áudio e 40 de vídeo. O k6 v2.3.0 executa uma iteração por VU. A stack local usa PostgreSQL com timestamps de commit, S3 e filas emulados e o adaptador de contrato do MediaConvert. A rodada de borda e a capacidade com o provedor real permanecem para homologação. [Especificação](../spec.md) · [Perfil e procedimento](../dev-infra/f10-story-005.md)

## Rodada inicial em dev

O serviço estava em `d7ac67d`, os testes em `c1782eb`, a infraestrutura em `0ec49e6`, a UI em `57f4e63` e o catálogo Liquibase em `ac0e851`. Comando `DOCKER_CONTEXT=colima-scrapbook-f10-validation TMPDIR=/private/tmp python3 -B mixed_submission_load.py --readiness --output <diretório-privado>`.

| Sinal | Resultado observado |
| --- | ---: |
| Iterações e pico de jornadas em voo | 200 e 200 |
| Eventos sintéticos de PUT registrados | 170 |
| Recibos F08 | 68 de 200 |
| p95 até o recibo dos 68 confirmados, posição por arredondamento para cima | 6,292 s |
| p95 de admissão | 468,585 ms |
| p95 da autorização de upload | 272,263 ms |
| p95 da conclusão do envio | 13,463 ms |
| p95 da consulta de recibo | 3,082 ms |

A leitura do PostgreSQL confirmou 132 notificações com instante anterior à admissão da sessão. O `LastModified` fornecido pelo emulador teve precisão de segundos. O banco registrou as 170 notificações, fixou 38 versões elegíveis e confirmou 19 arquivos em 18 registros de mídia. As 50 jornadas de texto completaram os 68 recibos. A amostra parcial impede a conclusão dos 95% para a coorte de referência.

O snapshot final comprovou o commit de disponibilidade dos 19 arquivos da coorte parcial, composto por quatro áudios, sete fotos e oito vídeos. Foram 190 amostras de comparação entre a VM e o relógio monotônico do host. A diferença entre as faixas de offset foi de 30.633,361 ms, superior aos 500 ms admitidos pelo runner. O processo encerrou com código 1 após a falha de checks do k6 e classificou a prontidão como inconclusiva por descontinuidade de relógio. Os recursos do ensaio foram removidos.

O reteste usa a correção exclusiva do modo local da F10. A notificação toma o relógio do banco após a observação de cada lote de versões PUT. Os processos e a validação de janelas da aplicação continuam usando o relógio real. [Correção revisada](https://github.com/diorgenesbackes/scrapbook-tests/pull/27)

## Reteste com o coletor corrigido

Em `1157ee1a4f424baa7e9fec816e2571e262796365` de testes, com os demais SHAs iguais, o k6 atingiu 200 jornadas em voo e confirmou os 200 recibos, 100 por evento, com 170 uploads. Passaram 890 checks. Houve zero erros HTTP em 1.505 chamadas. O p95 até o recibo foi de 13,832 s e o máximo foi de 14,606 s.

| Operação da API | p95 |
| --- | ---: |
| Admissão | 283,431 ms |
| Autorização de upload | 237,810 ms |
| Conclusão | 21,636 ms |
| Consulta de recibo | 7,647 ms |

Em uma consulta independente já com todos os 170 arquivos confirmados, 163 tinham commit de disponibilidade comprovado pelo PostgreSQL em até 72,750 s monotônicos desde antes da criação da stack. Esse intervalo começa antes de qualquer confirmação e termina depois da leitura de todos os commits observados, por isso é um limite superior do tempo após confirmação para esses 163 arquivos. A prova cobre 95,882% da coorte. Foram 39/40 áudios, 87/90 fotos e 37/40 vídeos, com 83/85 no evento A e 80/85 no B. O limite monotônico independe da diferença entre os relógios civis e não fornece percentis individuais. A consulta usou o mesmo SQL versionado do runner e não modificou o ambiente.

Uma segunda observação verificou 165 commits em até 139,827 s, ou 97,059% da coorte, com os 170 fatos publicados. Duas fotos ainda estavam em execução, e um áudio e dois vídeos, em reconciliação. O PostgreSQL rejeitou cinco atualizações de tentativas para `succeeded` pela constraint `ck_processing_attempts_state`. A regressão do relógio civil deixou `finished_at` entre 28,5 s e 30,6 s antes de `started_at` nas linhas rejeitadas.

Na última observação antes de 300 s, o PostgreSQL comprovou os 170 commits de disponibilidade em até 279,366 s monotônicos, contados a partir da mesma âncora anterior à criação da stack. Os 85 arquivos de cada evento estavam disponíveis, com todas as variantes requeridas no perfil.

O relatório padrão classificou o cálculo individual de `confirmedAt` até o commit como inconclusivo. Em 153 amostras de relógio, a diferença entre as faixas de offset chegou a 30.658,217 ms, acima dos 500 ms de tolerância. Os resultados de HTTP, a coorte completa e os limites monotônicos observados foram preservados. O ensaio de capacidade real, os percentis individuais da F10 e a rodada de borda permanecem pendentes para homologação.
