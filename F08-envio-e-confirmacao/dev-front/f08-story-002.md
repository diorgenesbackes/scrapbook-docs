# F08-STORY-002 — Envio e recibo de depoimento com foto

O fluxo reutiliza a revisão de foto e legenda da F07 e a concordância versionada da F08. Ao selecionar **Enviar**, calcula o SHA-256 em um worker, admite o manifesto de uma foto e obtém autorização PUT para a tentativa. A tela anuncia o progresso pelos bytes realmente transferidos e apresenta a validação do recebimento em uma etapa própria. [Contrato HTTP](../spec.md#5-contratos-da-api)

Uma interrupção preserva a tentativa em memória e permite retomar com a mesma chave e credencial. Após transferência, o navegador consulta o estado original, inclusive depois do fechamento da janela. Recibo, descarte e mudança de formato liberam o rascunho e as prévias. O recibo contém a confirmação e a data em Brasília. [Plano de aceite](../test-funcional/2026-11-25_14-54-00/1-plano.md)

A origem de upload exata é configurada no build. O Nginx de desenvolvimento aceita o endereço de loopback da fixture quando os diagnósticos locais estão habilitados. O pacote do Amplify recebe a origem HTTPS pela variável `SUBMISSION_UPLOAD_ORIGIN`. As rotas de armazenamento retornadas pela API precisam corresponder a essa origem para iniciar a transferência.
