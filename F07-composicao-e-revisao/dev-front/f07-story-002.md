# Seleção e revisão de fotos

O evento aberto oferece `/e/{code}/depoimento` e `/e/{code}/momento`. A seleção usa a janela da F05 e reaproveita a revisão, a identificação local da F06 e o diálogo de descarte da primeira entrega. Depoimento aceita uma foto e momento aceita uma a dez. [Card](https://trello.com/c/djeVwk02) · [PRD](../../PRD.md#f07-composição-e-revisão) · [Primeira história](f07-story-001.md)

Para o contrato da F08, **20 MB correspondem a 20.000.000 bytes por arquivo**. A contagem da legenda usa a mesma unidade de valores escalares Unicode proposta para o texto na primeira história. A legenda única é opcional, admite 500 caracteres e mostra o contador aos 100 restantes.

## Validação e recuperação

A aplicação verifica tamanho, assinatura e prévia decodificável antes de aceitar a foto. JPEG e PNG são reconhecidos pelos bytes iniciais. HEIC/HEIF usa as marcas da caixa `ftyp` e a capacidade nativa de decodificação do navegador. Arquivos rejeitados recebem feedback por arquivo e bloqueiam a revisão até o convidado descartar os rejeitados ou corrigir a seleção. Uma troca inválida conserva a foto anterior.

O navegador cria URLs `blob:` em memória para as prévias. A política do Nginx local e a configuração de publicação permitem essa origem apenas em `img-src`. Trocar, remover, descartar ou desmontar a página libera as URLs. Os arquivos e a legenda permanecem na página enquanto o rascunho existir. [S20](../../design-system/screens/S20.md) · [S24](../../design-system/screens/S24.md) · [S25](../../design-system/screens/S25.md)

## Contrato para a F08

| Campo | Valor |
| --- | --- |
| `eventCode` | Código público associado ao evento pela F05. |
| `category` | `testimonial` ou `moment`. |
| `format` | `photo`. |
| `photos` | Lista na ordem da revisão. Cada item contém o `File` original, o formato detectado e o tamanho em bytes. |
| `caption` | Legenda única, ou `null` quando ausente. |

A identificação confirmada pela F06 pode ser agregada somente com o mesmo `eventCode`. As URLs de prévia ficam fora do contrato. A F08 validará o conteúdo e o tamanho efetivamente recebidos e confirmará o conjunto. A câmera é tratada na F07-STORY-004. [Regras de mídia](../../guidelines/mensageria.md)
