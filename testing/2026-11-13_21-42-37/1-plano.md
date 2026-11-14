# Plano: PP-STORY-006

[História](https://trello.com/c/IV6uDrPq) · [FDD](../../fdd/FDD-Privacidade-Publica-do-Evento/FDD-Privacidade-Publica-do-Evento.md)

## Base e escopo

[Pull anterior à codificação](dependencias-e-base.json) nos seis projetos. Feature PP-006 criada da main; merge da dependência PP-005 somente na feature, sem promover main. Auditar e completar as 35 subtarefas: contrato/consulta, proteção de HTTP e telemetria, cliente/exports, proxy e validação integrada.

## Validação prevista

- Build/unitários backend e lint/build/unitários UI, sem pacote novo.
- Runner manual exclusivo public_privacy.py com Compose/volumes aleatórios e fixtures privadas explícitas. Nenhuma escrita no volume persistente.
- Mesmo evento antes/depois de texto confirmado e metadados de mídia privados, com igualdade do contrato salvo tempo técnico; immediate/scheduled e três estados de participação.
- Visitante sem identidade, responsável e terceiro sintéticos: contrato público igual; operações privadas existentes negadas sem vínculo. Cognito/galeria/mídia reais não implementados.
- Interceptor de comandos EF somente no probe de testes: SQL efetivo com quatro colunas públicas; ActivityListener/MeterListener em sinais próprios sem URL/código/identidade.
- Browser Chromium/WebKit: rede/DOM/storage, cookies omitidos, Referer ausente, redirects recusados, troca de rota e saída da gestão, QR/PNG/print digitais sem conteúdo privado.
- Logs reais API/Nginx com sucesso/erro e indisponibilidade, sem URI utilizável. Cache/CORS/headers direto e proxy.
- Regressões de erros/recuperação, compartilhamento, apresentação/perfil 4G e carga pública pertinentes. Registrar métricas reais, revisões, falhas e cleanup.

## Limites

Nenhum deploy/main merge da PP-006, DDL, upload, autenticação externa ou alteração AWS. Pendências PP-002-029/PP-003-034 mantidas. Leitor real, papel, dispositivos e telemetria externa não podem ser declarados aprovados por emulação.
