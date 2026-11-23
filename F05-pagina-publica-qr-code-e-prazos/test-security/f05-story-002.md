# Verificação defensiva da F05-STORY-002

O ensaio local com identidades sintéticas confirma o vínculo atual e a resposta mínima. A sessão ausente ou credencial inválida recebe `401`. A conta sem vínculo recebe `404 RESOURCE_NOT_FOUND` tanto para evento existente quanto para inexistente. A consulta bem-sucedida usa `no-store`. Os logs locais do serviço e da UI preservam o template da rota pública e omitem o código e os dados privados sintéticos. A página pública mantém `noindex, nofollow`, `no-referrer` e `X-Frame-Options: DENY` na stack descartável.

O teste unitário usa um alias para conferir a origem fixa, rejeita origens com protocolo, credenciais e caminhos indevidos e restringe o fallback a loopback. O workflow simulado confirma que uma origem da UI divergente barra a publicação antes do Amplify. O QR é gerado no navegador, sem serviço remoto. [Aceite local](../test-funcional/f05-story-002.md)

Os headers e a telemetria do hosting publicado, a sessão real da F02 e o domínio definitivo do piloto permanecem casos de validação no ambiente correspondente.
