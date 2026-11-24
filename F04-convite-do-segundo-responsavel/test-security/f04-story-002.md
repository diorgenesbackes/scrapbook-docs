# Verificação defensiva da prévia e do aceite

O ensaio descartável verificou identidade ausente, conta sem e-mail confirmado, origem recusada, segredo incorreto e token de outro ambiente. O proprietário foi recusado ao tentar aceitar seu próprio convite. O novo vínculo autorizou somente o evento do convite. A conta que perdeu a disputa recebeu o estado já usado e a leitura de outro evento continuou recusada.

Prévia e aceite receberam o token no corpo JSON e enviaram `no-store`. A auditoria e os logs dos três serviços de teste omitiram a credencial sintética. O teste do codec confirma seleção de chave anterior, recusa de token malformado e diferença nos hashes entre conjuntos de chaves. A falha simulada de auditoria preservou o convite pendente e a única vaga ocupada.

O fluxo usa identidades de Development. A integração F02 cobre as condições da sessão real. A validação do hosting publicado deve confirmar os headers e a configuração de chaves do ambiente.
