# Verificação defensiva da emissão de convite

HTTP e PostgreSQL confirmaram que a conta sem vínculo recebe o mesmo `404` de um evento inexistente, a rota autenticada recusa identidade ausente e o POST recusa origem divergente. A falha de auditoria simulada reverte a emissão e restaura o pendente anterior.

O codec recupera um segredo cifrado válido e recusa alteração do conteúdo, da operação, do evento ou da chave do ambiente. A tabela mantém hash indexado e ciphertext pendente. A substituição e o vencimento materializado zeram o ciphertext e sua referência de chave. Logs da UI e API e dados da auditoria ficam sem o token sintético.

A página de entrada entrega `no-referrer` e `noindex, nofollow` no Nginx local. A aplicação remove o fragmento após abrir o endereço. A validação do hosting publicado e das chaves do ambiente faz parte do aceite em homologação.
