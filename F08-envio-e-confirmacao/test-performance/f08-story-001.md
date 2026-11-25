# Prontidão e confirmação sob carga local

Em 25/11/2026, os candidatos integrados `b0ede23` (API), `9c4a570` (UI) e `1623a1b` (testes) foram medidos em Docker com eventos e dados sintéticos. [Plano de aceite](../test-funcional/2026-11-25_10-24-40/1-plano.md)

| Perfil | Resultado | Critério |
| --- | --- | --- |
| k6, 200 convidados simultâneos com uma confirmação cada, distribuídos em dois eventos | p95 **207,29 ms**, 200 de 200 confirmações e zero falhas HTTP. A consulta ao banco verificou 200 novos registros e um primeiro fato por evento. | p95 ≤ 500 ms e 200 confirmações válidas. |
| Android emulado, cinco aberturas com cache frio | **1.990, 1.985, 1.986, 1.985 e 1.989 ms** até o editor estar disponível. | Prontidão ≤ 3.000 ms. |
| Android emulado, envio até o recibo | **213, 214, 211, 198 e 198 ms**. | Medição descritiva. |

A rede do navegador foi emulada com RTT de 150 ms, download de 1,6 Mbit/s e upload de 750 kbit/s, sem throttling de CPU. O k6 mediu o POST de texto em loopback com consentimentos emitidos no preparo. Latência externa e transferência de mídia ficam fora desse perfil. [Especificação dos testes](../spec.md#7-estratégia-de-testes)
