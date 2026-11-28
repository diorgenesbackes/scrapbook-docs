# Plano de implementação da F11 — Galeria e reprodução

A entrega abrange serviço, frontend, infraestrutura e testes. A especificação reúne os contratos, o comportamento esperado e a matriz de aceite de cada capacidade. [Especificação](spec.md)

## Pré-requisitos

- Partir das entregas integradas da F03, F08 e F10. Conferir a rota de lembranças, o contrato de abertura, a confirmação canônica e o perfil ativo de cada variante. [F03](../F03-eventos-e-politica-de-abertura/spec.md) · [F08](../F08-envio-e-confirmacao/spec.md) · [F10](../F10-processamento-de-midias/spec.md)
- Usar o ambiente descartável do projeto e os SDKs e gerenciadores fixados pelos manifests. Preparar eventos sintéticos aberto e fechado, duas contas responsáveis, uma conta sem vínculo, contribuições de todos os tipos e variantes em estados diferentes. [Ambiente e base técnica](../guidelines/README.md)
- Para homologação, confirmar publicação e ativação da F10, identidade de responsável válida e recursos privados de entrega de mídia. A preparação local pode avançar com o laboratório. [Entrega da F10](../F10-processamento-de-midias/delivery/f10-story-005.md)

## Fase 1 — Abrir a galeria autorizada

**1. Acesso à área de lembranças.** Integrar a galeria à superfície da F03 e aos estados de abertura, falta de vínculo e sessão inválida. Garantir que evento fechado só mostre data e contagem regressiva.

**2. Página cronológica.** Entregar a lista de contribuições confirmadas, permitindo percorrer o evento do primeiro recebimento ao último e usar os filtros de categoria.

**3. Páginas e conjuntos.** Preservar o desempate, a posição dos arquivos, a autoria e a legenda quando um conjunto atravessar páginas. Observar o custo da leitura no PostgreSQL conforme o modelo existente.

**4. Estados da galeria.** Mostrar carregamento, vazio, falha, mídia em preparação e mídia temporariamente indisponível quando o acesso estiver liberado.

## Fase 2 — Exibir a lembrança com mídia privada

**5. Detalhe de uma contribuição.** Entregar a abertura individual de depoimentos e momentos com os dados autorizados. Manter a seleção vinculada ao evento e à sequência da galeria.

**6. Caminho privado das variantes.** Preparar a configuração de entrega de mídia por ambiente e a emissão temporária para a variante pronta do perfil ativo. Demonstrar separação entre capas públicas e lembranças privadas.

**7. Miniaturas e fotos.** Exibir as miniaturas autorizadas na galeria, abrir a foto no visualizador e preservar a proporção. Tratar ausência e vencimento sem recorrer ao original.

**8. Reprodução de áudio e vídeo.** Permitir reprodução, pausa e busca de posição pelos controles da interface. Preservar o item atual ao terminar e oferecer nova autorização quando a transferência precisar ser retomada.

## Fase 3 — Percorrer a sequência e reagir à perda de acesso

**9. Navegação em sequência.** Entregar Anterior, Próximo e Voltar, inclusive nas bordas entre páginas. Preservar filtro, posição e foco durante a navegação autorizada.

**10. Fechamento e perda de vínculo.** Interromper a consulta e a reprodução ao detectar perda de acesso. Remover conteúdo da superfície e descartar respostas atrasadas mesmo com mais de uma aba em uso.

**11. Retomada e acessibilidade.** Garantir revalidação no foco e no retorno à página, navegação por teclado, foco previsível e anúncios de estado. Exercitar as referências S28, S29 e S30 em diferentes larguras de tela. [Design system](../design-system/screen-matrix.md)

## Fase 4 — Preparar a entrega no ambiente publicado

**12. Desempenho e isolamento.** Medir as operações comuns sobre uma galeria representativa. Verificar o isolamento entre eventos, a exposição anônima e a ausência de conteúdo ou credenciais nos sinais operacionais.

**13. Ativação e renovação no ambiente.** Conferir o caminho privado com as variantes reais da F10, incluindo a renovação após expiração, a busca de posição e o bloqueio durante a reprodução.

**14. Continuidade entre réplicas.** Conferir que a próxima página funciona ao alternar réplicas e que a rotação da chave de entrega preserva somente o prazo das autorizações vigentes.

**15. Entrega operacional.** Registrar parâmetros de publicação, comportamento após fechamento e forma de desativar a entrega privada diante de problema. Registrar as evidências locais e de homologação e os pontos pendentes por ambiente. [Estratégia de testes](spec.md#7-estratégia-de-testes)
