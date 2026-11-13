# Handoff HTML/CSS

[Índice](README.md) · [Tokens](tokens.css) · [Componentes](components.css)

Importar tokens.css antes de components.css. catalog.css e catalog.js pertencem ao ambiente de revisão, não à aplicação. Classes sb-* são independentes de React. Usar os comportamentos de componentes/acessibilidade da especificação ao conectar o backend.

## Formulário e composição

```html
<link rel="stylesheet" href="tokens.css">
<link rel="stylesheet" href="components.css">
<main class="sb-shell">
  <div class="sb-form-layout sb-stack">
    <h1>Criar evento</h1>
    <form class="sb-stack" aria-describedby="form-help">
      <p id="form-help">Preencha os dados do evento. Os campos abaixo são obrigatórios.</p>
      <div class="sb-field">
        <label for="event-name">Nome do evento</label>
        <input id="event-name" name="name" class="sb-input"
          required aria-describedby="event-name-help">
        <p id="event-name-help" class="sb-help">Entre 1 e 120 caracteres.</p>
      </div>
      <button type="submit" class="sb-button">Criar evento</button>
      <p role="status" aria-live="polite"></p>
    </form>
  </div>
</main>
```

Na falha de validação, manter o valor, acrescentar aria-invalid=true e associar o erro por aria-describedby. Exemplo de erro: `<p id="event-name-error" class="sb-error-text">Informe o nome do evento.</p>`. Focar o primeiro inválido ou um resumo navegável. Durante escrita, usar aria-busy=true no form, desabilitar o submit e trocar o texto por “Salvando…”. Depois de conflito/resposta perdida, não habilitar repetição sem reconciliação.

## Card autorizado

```html
<a class="sb-card sb-card--interactive" href="/events/123">
  <img class="sb-photo" src="/authorized-cover.webp" width="640" height="480"
    loading="lazy" alt="Capa do evento Festa no jardim">
  <div class="sb-card-body">
    <h2>Festa no jardim</h2>
    <p class="sb-help">15 de outubro de 2027 · Brasília</p>
  </div>
</a>
```

Os caminhos acima ilustram a integração, sem definir URLs reais. Não aninhar botões no link do card. Capa pública e mídia privada são contratos diferentes. Manter srcset/sizes e dimensões; produzir variantes otimizadas, preservando original privado. Proposta de orçamento para implementação: logo SVG ≤10KiB; capa/hero responsivo ≤200KiB no mobile; miniatura ≤80KiB, a validar com qualidade e perfil 4G. Os PNGs de referência não cumprem esse orçamento e não devem ser copiados sem otimização ao bundle de produção.

## Bloqueio temporal

```html
<section class="sb-empty" aria-labelledby="locked-title">
  <h1 id="locked-title">Cada lembrança tem seu tempo.</h1>
  <p>Disponível em 15 de outubro de 2028, à meia-noite de Brasília.</p>
  <p aria-label="Faltam 365 dias, 8 horas e 24 minutos">365 dias · 08 horas · 24 minutos</p>
</section>
```

Não montar galeria, count, remetentes ou URLs para ocultá-los por CSS. Atualizar contagem pela referência temporal do servidor e consultar autorização antes da transição; nenhum timer local concede acesso.

## Progresso e confirmação

```html
<label for="upload">Enviando arquivos — 64%</label>
<progress id="upload" class="sb-progress" value="64" max="100">64%</progress>
<p role="status">Transferência em andamento.</p>
<button class="sb-button" disabled aria-busy="true">Enviando…</button>
```

Na validação posterior, retirar value e informar “Conferindo seu envio.”; só após persistência confirmada mostrar o recibo sem prévia. Retry mantém a mesma tentativa e conteúdo em memória, dentro do prazo. Não acrescentar Enviar novo durante pendência.

## Diálogo e abas

Usar `<dialog aria-labelledby="..." aria-describedby="...">` com showModal(), ação segura autofocus e form method=dialog; Escape/cancel não executam descarte. Guardar o acionador e devolver foco no close. Para salvar algo assíncrono, impedir fechamento automático até concluir e manter erro acessível.

Abas precisam de role=tablist/tab/tabpanel, aria-controls/aria-labelledby, aria-selected, um tabindex=0 e os demais -1. Setas/Home/End movem/selecionam; trocar filtros reinicia o cursor e carrega somente dados autorizados. O exemplo interativo de catalog.js demonstra a interação de foco/seleção; não implementa consulta real de galeria.

## Impressão e e-mail

S15 usa uma página de impressão com nome/link legível e QR real sobre branco. O protótipo tem slot claramente rotulado, não um código válido. Preservar quiet zone exigida pelo gerador e validar leitura em papel/aparelhos antes do aceite. Nunca oferecer download das lembranças.

[email-example.html](email-example.html) demonstra estrutura com tabela de apresentação, CSS inline e preheader. Não incluir conteúdo/quantidade/remetentes de contribuições. Recuperação/confirmar endereço são templates gerenciados Cognito sujeitos à Q01; e-mails de convite/liberação seguem os mesmos tokens com fallback de fonte do cliente.
