# Desempenho da página pública

O ensaio local usa a stack descartável do [runner](../../../scrapbook-tests/event_public_stack.py). O k6 simula 200 convidados em dois eventos, com três GETs por convidado. A página é medida separadamente da navegação à capa carregada e ao estado da janela visível. Chromium emula Pixel 7, cache frio, RTT 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s. Metas da story de p95 até 500 ms e página pronta em até 3 s. [Critérios](../spec.md#aceite-e-integração)

| Execução | GETs | p95 | Página em 4G |
| --- | ---: | ---: | ---: |
| Inicial | 595 de 600 | 112,44 ms | Medição executada na etapa de diagnóstico |
| Diagnóstico | 595 de 600, cinco respostas `500` | 103,72 ms | 2.141 ms |
| Após corrigir o pool, na branch da story | 600 de 600 | 73,81 ms | 2.091 ms |
| Revalidação com o resultado integrado em `dev` | 600 de 600 | 68,77 ms | 1.838 ms |
| Revalidação em `dev` após a correção de navegação | 600 de 600 | 62,16 ms | 2.089 ms |

No diagnóstico o PostgreSQL registrou cinco recusas por limite de slots. A correção limita o pool e o reteste preserva o mesmo perfil e os mesmos limiares. [Configuração e pressão de conexões](../dev-back/f05-story-001.md)

A medição cobre consultas da página. O ensaio integrado de envios e a calibração do WAF com convidados atrás do mesmo IP são verificados nos ambientes e features correspondentes.
