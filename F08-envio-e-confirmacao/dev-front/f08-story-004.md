# F08-STORY-004 — Envio de vídeos na jornada do convidado

O convidado pode enviar o vídeo selecionado ou gravado na revisão S25. O worker lê até 256 KiB por vez e calcula o SHA-256 integral e os digests das partes. A transferência passa a usar partes de 16 MiB a partir de 32 MiB. O progresso de S26 soma as partes que o armazenamento confirma e os bytes enviados na parte corrente. Após uma queda, o cliente preserva a tentativa e o arquivo em memória e consulta as partes pendentes. A tela S27 aparece a partir da confirmação com horário do servidor. [Spec da F08](../spec.md)

A implementação está em `3b77dc1` e a suíte independente em `5a5bf90`. Lint, build e 171 testes unitários passaram com Node 26.13.0 e npm 11.20.0. As 36 novas verificações cobrem manifesto, limites, hashing, falhas recuperáveis, progresso e recibo.
