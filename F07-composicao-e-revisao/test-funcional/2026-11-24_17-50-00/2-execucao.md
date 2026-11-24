# Execução do aceite da F07-STORY-003

O ensaio de 24/11/2026 passou na stack descartável sobre `dev`. As jornadas usaram a consulta real da F05 e o contexto de identificação sintético da F06. [Plano](1-plano.md) · [Evidência de navegador](../../test-e2e/f07-story-003.md)

| Critério | Evidência | Resultado |
| --- | --- | --- |
| CA1 | Vídeo apareceu nas duas categorias e áudio somente em depoimento. A revisão aceitou vídeo de 60 s em momento e vídeo e áudio de 300 s em depoimento. As amostras de 61 e 301 s foram recusadas nas categorias correspondentes. Os unitários verificaram os limites de 50.000.000 e 500.000.000 bytes e a rejeição de um byte acima de cada limite. | Aprovado |
| CA2 | As oito combinações de container e codec exibiram prévia nos cinco perfis. Arquivos vazios, corrompidos, incompatíveis e disfarçados bloquearam a revisão e permitiram nova seleção. | Aprovado |
| CA3 | A revisão disponibilizou o player com reprodução explícita, legenda e retorno à edição. A troca bem-sucedida atualizou a prévia. Os unitários verificaram o contrato com arquivo, container, tamanho, duração, legenda e o código do evento. | Aprovado |
| CA4 | A legenda e a seleção válida foram preservadas após falha de troca. Voltar à edição pausou a reprodução. Troca, remoção e descarte liberaram as URLs. Um segundo evento iniciou com estado independente. | Aprovado |

Os ensaios locais usaram emulação para os perfis móveis. A revisão trafegou zero escritas HTTP e o runner verificou a ausência de contribuições no banco. [Segurança](../../test-security/f07-story-003.md)
