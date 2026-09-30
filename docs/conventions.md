# Convenções

## Nomenclatura de classes CSS

O projeto não adota BEM nem utilitários. Usa nomes curtos, em inglês, **por papel na
interface**, com prefixo de bloco quando o elemento só existe dentro de um componente.

```
.role-card          bloco
.role-card .rc-top     descendente, prefixo abreviado do bloco
.role-card .rc-co
.role-card .rc-when
.role-card .rc-tags
```

Regras que valem para toda a folha:

| Regra | Exemplo |
| --- | --- |
| Blocos em `kebab-case` | `.logo-chip`, `.contact-rail`, `.band-label` |
| Descendentes com prefixo abreviado do bloco | `.rc-` em `.role-card`, `.cf-` em `.cfield`, `.nl-` em `.netlinks`, `.m-` em `.mini` |
| Estado como classe adjetiva, sem prefixo | `.in`, `.typing`, `.full`, `.primary`, `.ghost` |
| Capacidade detectada como classe na raiz | `.js-reveal` |
| Papel semântico vence aparência | `.muted` descreveria a cor; o token `--muted` já faz isso, então a classe é `.band-sub` |
| Abreviação de uma letra só em contexto denso | `.k` `.v` `.d` em `.stat`, `.ln` `.lc` no visualizador de código, `.c` no comentário de rótulo |

Classes de destaque de sintaxe usam a abreviação léxica convencional de editores:
`.kw` palavra-chave, `.st` string, `.cm` comentário, `.nm` nome, `.nu` número, `.dn` dunder,
`.doc` docstring.

### Ordem da folha de estilos

```
1.  :root                  tokens, precedidos do comentário que descreve o conceito de layout
2.  reset e base           box-sizing, html/body, a, ::selection, :focus-visible
3.  chrome                 .shell, .topbar, .nav, .progress, .footer
4.  vocabulário comum      .eyebrow, .panel-label, .card, .tag
5.  por tela               início → sobre-mim → contato, na ordem da navegação
6.  revelação              .js-reveal, .reveal, .stagger, .words
7.  @media                 1100px → 820px → prefers-reduced-motion
```

Consultas de mídia ficam agrupadas no fim, não junto de cada componente. Com apenas dois
breakpoints, a leitura fica mais clara ao ver todas as adaptações juntas.

### Especificidade

Seletor de classe simples é o padrão. Descendência só quando o elemento depende do contexto
(`.js-reveal .reveal.in`). Sem `#id` como gancho de estilo, sem `!important` — com uma
exceção documentada: `[hidden]{display:none !important}`, necessária porque as seções
declaram `display: flex` e uma regra de autor vence a do agente do usuário.

## Nomenclatura em JavaScript

O código do script é escrito **em português**, para ficar coerente com o conteúdo do site e
com os comentários. Os identificadores da plataforma permanecem como são.

```js
var garantirVisiveis = function () { … };
function typeIn(el)   { … }   // nome técnico, mantido em inglês
function progresso()  { … }
function countUp(el)  { … }
```

| Regra | |
| --- | --- |
| `camelCase` para variáveis e funções | `navBtns`, `garantirVisiveis`, `countUp` |
| `SCREAMING_SNAKE_CASE` para constante de módulo | `ORDER` |
| `var` e expressões de função | Ver a justificativa em [`tech-stack.md`](tech-stack.md#linguagens) |
| Uma IIFE envolvendo tudo | Nenhuma variável vaza para o escopo global |
| Vínculo por `data-*`, nunca por classe de estilo | `data-go`, `data-scroll`, `data-file`, `data-pane`, `data-count`, `data-copy` |
| Guarda de idempotência com `dataset` | `if (el.dataset.done) return;` em `typeIn` e `countUp` |

A separação entre `data-*` e classe é a regra que mais importa na manutenção: renomear uma
classe é uma mudança de estilo e não pode quebrar comportamento.

## Nomes de arquivo

| Escopo | Convenção | Exemplos |
| --- | --- | --- |
| Arquivos especiais do GitHub, na raiz | `SCREAMING_CASE` | `README.md`, `SECURITY.md`, `LICENSE` |
| Documentação em `docs/` | `kebab-case`, em inglês | `architecture.md`, `design-system.md`, `tech-stack.md`, `conventions.md` |
| Arquivos servidos | minúsculas | `index.html`, `robots.txt`, `sitemap.xml` |
| Conteúdo dos documentos | Português | — |

Nomes de arquivo em inglês e conteúdo em português é intencional: o nome é endereço e
aparece em URL, link e árvore de arquivos; o conteúdo é para ler.

## Markup

- Elementos semânticos onde existem: `<header>`, `<main>`, `<section>`, `<aside>`,
  `<article>`, `<footer>`, `<nav>`
- Toda seção de tela tem `aria-label`; o item de navegação ativo tem `aria-current="page"`
- Todo link de ícone tem `aria-label`; todo SVG decorativo fica fora da árvore de
  acessibilidade
- A cópia duplicada dos logotipos, que existe só para fechar o loop da esteira, é
  `aria-hidden="true"` com `alt=""`
- Todo link externo leva `target="_blank" rel="noopener"`
- Todo controle de formulário tem `id` estável e `<label for>` correspondente
- Visibilidade é alternada pela propriedade `hidden`, nunca por `style.display`

## Commits

Mensagem no imperativo, em português, primeira linha até 72 caracteres, sem ponto final.

```
Publica o site pessoal
Atualiza canonical e Open Graph para o dominio novo
Adiciona documentacao, licenca, politica de seguranca e CSP por hash
```

Não há convenção de prefixo tipo `feat:` ou `fix:` — o repositório tem um único autor e um
único artefato, e o ganho de categorizar não pagaria o ruído.

## Alterando o `index.html`

1. Localize a região pelo mapa em [`architecture.md`](architecture.md#composição-do-documento)
2. Se tocou em `<style>` ou `<script>`, **recalcule os hashes da CSP**
   ([procedimento](../SECURITY.md#recalculando-os-hashes-da-csp))
3. Abra o arquivo, pressione F12 e recarregue: o console não pode ter nenhuma linha
   começando com `Refused to`
4. Verifique as três telas e as duas larguras de quebra (1100px e 820px)
5. Confirme que a página não rola na horizontal em nenhuma largura
6. Commit e push; o GitHub Pages republica sozinho
