# Decisões abertas e revisão de produto

[Índice](README.md) · [Prévia](index.html#decisions)

Escolha registrada do responsável: **fotográfico minimalista**. Demais decisões abaixo precisam de revisão antes do aceite final. A implementação atual da aplicação não é alterada por este handoff.

| ID | Questão / decisão necessária | Responsável | Impacto e tratamento provisório |
| --- | --- | --- | --- |
| Q01 | Comprovar branding, português, acessibilidade, regra de senha e cobertura de edição/confirmação nas telas gerenciadas Cognito. | Produto + desenvolvimento | S02–S08/S32 são referência; não substituir por login próprio nem prometer layout arbitrário. |
| Q02 | Aprovar formatos/peso/proporção de capa personalizada, limites e comportamento ao trocar/restaurar. | Produto | S12 propõe moldura 16:9, sem editor/crop; limites não inventados. |
| Q03 | Aprovar aviso final de acesso, privacidade/menores, retenção e canal fora da UI para pedidos. | Produto + revisão de privacidade | Texto provisório em S17; sem inventar ações de exclusão na versão inicial. |
| Q04 | Definir domínio, remetente e canal de suporte. | Produto/operação | example.invalid e nomes sintéticos; nenhum link de QR é escaneável no protótipo. |
| Q05 | Definir meta de conclusão sem ajuda após testes, leitor de tela/aparelhos reais e acessibilidade da mídia. | Produto + participantes | Validação local parcial registrada; não afirmar conformidade/qualidade de uso global. |
| Q06 | Confirmar rotas/retornos das superfícies derivadas: callback, confirmação, sessão expirada, saída, conflitos e resposta perdida. | Produto + FDDs | Inventário explicita inferências, sem mudar contratos existentes. |
| Q07 | Aprovar marca, fotos de referência, proporção de espaço/mídia e sistema visual. | Responsável do produto | Direção escolhida; proposta 0.1 ainda não aprovada em todas as telas. |
| Q08 | Alinhar a unidade de caracteres dos futuros textos/legendas com o backend. | Desenvolvimento + FDD de envios | Protótipo conta valores escalares Unicode; contrato definitivo precisa concordar. |

## Conflitos já resolvidos por decisão anterior

Aplicam-se os adendos aprovados do HLD: confirmação de e-mail; código Cognito de uma hora; travamento eventual por evento; URLs anteriores até dois minutos. A autenticação real continua adiada para implementação. As telas são especificadas para que a identidade cubra o produto completo.

## Superfícies que o PRD não nomeia como páginas

O PRD descreve fluxos, não um inventário exaustivo de rotas. Foram derivados: S04 confirmação, S08 callback, S12 capa, S17 identificação separável, S24 legenda, S26 estados de envio, S30 player, S31–S34 mensagens e S35 suporte/indisponibilidade; D01–D03 são diálogos. Estas inferências estão sujeitas a Q06 e não autorizam novas capacidades. FR-020/022 não exigem painel administrativo de lembranças: a operação fica nas ferramentas externas e alertas sem conteúdo.

## Registro de aceite

- Direção geral: escolhida pelo responsável em 2026-11-13.
- Revisão técnica do catálogo: ver ciclo de testes vinculado no índice principal.
- Revisão de todas as telas pelo produto: **pendente**.
- Questões Q01–Q08 resolvidas: **pendente**.
- Autorização de implementação/publicação da identidade completa: **não registrada**.

Trello To Deploy representa entrega técnica para revisão; não marcar Done nem declarar aprovação humana automaticamente.
