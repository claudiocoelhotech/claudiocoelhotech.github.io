# Design system

Sistema visual derivado da identidade usada nos conteúdos de
[@claudiocoelho.tech](https://www.instagram.com/claudiocoelho.tech/): base carvão quente,
laranja como única cor de destaque, monoespaçada nos elementos de interface e apresentação
do conteúdo textual como código Python.

Todos os valores vivem como custom properties no bloco `:root` no topo da folha de estilos
embutida. Nenhum componente declara cor literal — toda cor é lida de um token.

---

## 1. Cor

### Tokens

| Token | Valor | Papel |
| --- | --- | --- |
| `--bg` | `#171310` | Fundo da aplicação. Carvão com viés quente, não preto neutro |
| `--bg-0` | `rgba(23,19,16,0)` | Mesmo `--bg` com alfa zero, para degradês sem esmaecimento acinzentado |
| `--surface` | `#201B17` | Superfície elevada: cartões, campos de formulário, cabeçalho |
| `--surface-2` | `#2A2420` | Segundo nível de elevação: etiquetas dentro de cartões |
| `--line` | `#362E28` | Borda de componente |
| `--line-soft` | `#282119` | Divisor estrutural entre regiões e faixas |
| `--fg` | `#F4EFE7` | Texto primário. Creme, não branco puro |
| `--muted` | `#A0948A` | Texto secundário, rótulos, texto de apoio |
| `--accent` | `#EE6B1F` | Destaque único do sistema |
| `--accent-soft` | `#F7A05A` | Estado de hover do destaque e realce em bloco de código |
| `--chip` | `#E9E2D6` | Ladrilho creme dos logotipos, único plano claro da interface |

### Contraste medido

Razões calculadas pela fórmula de luminância relativa da WCAG 2.1. O mínimo AA para texto
normal é 4.5:1; para texto grande (≥ 24px ou ≥ 19px em negrito), 3:1.

| Par | Razão | Conformidade |
| --- | --- | --- |
| `--fg` sobre `--bg` | 16.14:1 | AAA |
| `--fg` sobre `--surface` | 14.91:1 | AAA |
| `--muted` sobre `--bg` | 6.24:1 | AA |
| `--muted` sobre `--surface` | 5.77:1 | AA |
| `--accent` sobre `--bg` | 5.94:1 | AA |
| `--accent` sobre `--surface` | 5.49:1 | AA |
| `--accent-soft` sobre `--bg` | 8.91:1 | AAA |
| Texto do código (`#B8ABA0`) | 8.24:1 | AAA |
| Docstring e comentário (`#8E8279`) | 4.94:1 | AA |
| String no código (`#7FD1A0`) | 10.14:1 | AAA |
| Palavra-chave (`#C77BE8`) | 6.53:1 | AA |
| Número e dunder (`#7FB7E8`) | 8.65:1 | AAA |
| Texto escuro sobre `--chip` | 14.45:1 | AAA |

Nenhum par em uso reprova em AA. O `--accent` passa em AA inclusive para texto normal,
o que permite usá-lo em rótulos pequenos sem exceção.

### Regras de aplicação

- **Um único destaque.** O laranja marca o elemento que carrega a informação, nunca a
  decoração. Em cada bloco há no máximo um uso.
- **Neutros com viés.** Todos os cinzas carregam componente quente (R > G > B), alinhados
  ao destaque. Cinza neutro puro leria como inacabado ao lado do laranja.
- **Tema único.** O sistema não implementa tema claro. A folha de estilos declara
  `color-scheme: dark` em `:root` e define explicitamente fundo e cor de todo elemento,
  para que a página se comporte igual sobre qualquer plano de fundo do hospedeiro.

---

## 2. Tipografia

### Famílias

| Token | Pilha | Uso |
| --- | --- | --- |
| `--font-display` | `'Archivo', 'Helvetica Neue', Arial, sans-serif` | Títulos, números de destaque |
| `--font-body` | `'Archivo', system-ui, -apple-system, sans-serif` | Texto corrido |
| `--font-mono` | `'JetBrains Mono', ui-monospace, SFMono-Regular, Menlo, monospace` | Rótulos de interface, navegação, código, dados tabulares |

Duas famílias em três papéis. A Archivo é uma grotesca de baixo contraste que sustenta pesos
altos sem fechar as contraformas; a JetBrains Mono carrega a camada de interface e reforça a
leitura de "ambiente de desenvolvimento" que a página propõe.

Pesos carregados: Archivo 400, 500, 600, 700, 800; JetBrains Mono 400, 500, 700.
Ambas com `display=swap`, de modo que o texto renderiza imediatamente na fonte de fallback.

### Escala

A escala é fluida — os tamanhos maiores interpolam por `clamp()` em vez de saltar em
breakpoints. Dezoito declarações `clamp()` ao longo da folha.

| Elemento | Tamanho | Observações |
| --- | --- | --- |
| `h1` | `clamp(2.6rem, 7.5vw, 4.8rem)` | `line-height: .98`, `letter-spacing: -.03em`, `text-wrap: balance` |
| `h2` de faixa | `clamp(1.7rem, 4.2vw, 2.5rem)` | `line-height: 1.08`, `letter-spacing: -.025em` |
| Valor de estatística | `clamp(1.9rem, 4.6vw, 2.7rem)` | `font-variant-numeric: tabular-nums` |
| Título de cartão | `16px` / 700 | |
| Corpo | `15px` / `line-height: 1.6` | |
| Texto de cartão | `13.5px` – `14px` / `line-height: 1.6` | |
| Código | `13px` / `line-height: 1.85` | |
| Numeração de linha | `11.5px` | `tabular-nums`, `user-select: none` |
| Rótulo de interface | `10.5px` – `12px` | maiúsculas, `letter-spacing: .14em` – `.16em` |

Regra de espaçamento entre letras: quanto menor e mais em caixa alta, maior o tracking;
quanto maior o título, mais negativo. Títulos grandes chegam a `-.03em`, rótulos a `+.16em`.

### Uso da monoespaçada

A monoespaçada não é ornamento. Ela marca o que é **dado** e não prosa: navegação,
rótulos de campo, contatos, blocos de código, etiquetas, contador de tela e comentários
de seção. A proporcional fica com títulos e texto corrido. A fronteira entre as duas
é a fronteira entre interface e conteúdo.

---

## 3. Espaçamento e layout

### Grade

Largura máxima de leitura de 1040px (`.wrap`), centrada, com gutter lateral mínimo de 20px
em qualquer viewport. As telas em modo editor abrem em grade assimétrica:

| Tela | Colunas (desktop) |
| --- | --- |
| `_sobre-mim` | `minmax(0,1.3fr)` editor + `minmax(0,1fr)` painel de cases |
| `_contato` | `minmax(0,1fr)` contatos + `minmax(0,1.02fr)` formulário |

Todo filho de grade que contém texto, código ou tabela recebe `min-width: 0`, para que o
conteúdo quebre dentro da coluna em vez de empurrar a página.

### Breakpoints

Apenas dois, mais uma consulta de preferência. A escala fluida absorve o resto.

| Consulta | Efeito |
| --- | --- |
| `max-width: 1100px` | Painel lateral passa a ocupar a largura total abaixo do conteúdo |
| `max-width: 820px` | Todas as grades colapsam para uma coluna; a navegação passa a ocupar a linha inteira |
| `prefers-reduced-motion: reduce` | Desliga animações e transições e força o estado final de todo elemento revelável |

### Respiro vertical

Faixas usam `padding: clamp(44px, 5.5vw, 70px) 20px`. Regiões de editor usam
`clamp(26px, 3.4vw, 44px)`. O ritmo vertical escala com a viewport em vez de alternar
entre dois valores fixos.

### Área segura

A página respeita `env(safe-area-inset-*)`: o cabeçalho fixo usa
`top: env(safe-area-inset-top, 0px)` e o rodapé soma `env(safe-area-inset-bottom, 0px)`
ao próprio padding, de modo que o conteúdo não fica sob as barras do sistema em telas
com recorte.

---

## 4. Componentes

| Componente | Classe | Descrição |
| --- | --- | --- |
| Barra superior | `.topbar` | Cabeçalho fixo com marca, navegação, selo de status e barra de progresso |
| Barra de progresso | `.progress` | Filete laranja de 2px no rodapé do cabeçalho, largura proporcional à rolagem |
| Navegação | `.nav button` | Rótulos monoespaçados com prefixo `_`; item ativo recebe sublinhado laranja e `aria-current` |
| Selo de status | `.status` | Ponto verde com halo e texto em caixa alta |
| Sobrancelha | `.eyebrow` | Rótulo laranja em caixa alta precedido de quadrado sólido de 8px |
| Rótulo de faixa | `.band-label` | Comentário Python `#` seguido de texto datilografado com cursor |
| Cartão de estatística | `.stat` | Rótulo, valor grande em laranja com `tabular-nums` e descrição |
| Etiqueta | `.tag`, `.rc-tag` | Pílula de borda, monoespaçada, para competências e ferramentas |
| Ladrilho de logotipo | `.logo-chip` | 132×74px, fundo creme, logotipo em escala de cinza; a cor retorna no hover |
| Esteira | `.logo-marquee` | Faixa de largura total com cortinas de degradê nas bordas e trilha em loop |
| Cartão de case | `.role-card` | Cabeçalho de metadados, título com barra vertical, parágrafos, etiquetas e link externo |
| Visualizador de código | `.code` | Grade de duas colunas: medianiz de numeração + linha, com destaque de sintaxe |
| Campo de contato | `.cfield` | Rótulo com ícone, valor em 16px e botão de cópia |
| Link de rede | `.netlinks a` | Ícone, nome, identificador alinhado à direita e seta externa |
| Campo de formulário | `.field` | Rótulo em caixa alta espaçada + controle com borda que acende em foco |
| Botão | `.btn.primary`, `.btn.ghost`, `.submit` | |

### Elevação

Três planos, sem sombra projetada. A separação é feita por diferença de luminância e por
borda de 1px:

```
--bg (#171310)  →  --surface (#201B17)  →  --surface-2 (#2A2420)
```

Sombra só aparece no ladrilho de logotipo, para assentar o plano creme sobre o escuro.

### Raio de borda

`6px` em controles pequenos, `9px` em campos e botões, `10–12px` em cartões e ladrilhos,
`999px` em etiquetas. O raio cresce com a área do elemento; nada usa um valor único global.

---

## 5. Motion

Seis conjuntos de `@keyframes`. Todos param sob `prefers-reduced-motion: reduce`.

| Nome | Alvo | Função |
| --- | --- | --- |
| `rise` | Elementos do hero | Entrada em cascata, deslocamento de 16px com atraso escalonado de 40ms a 520ms |
| `drift-a`, `drift-b` | Halos de fundo | Deriva ambiente de 24s e 29s, em `alternate`, com fases distintas para não sincronizar |
| `blink` | Cursor `>` da linha de papéis | Piscada de terminal, `steps(1)` a cada 1.15s |
| `bob` | Seta da chamada de rolagem | Oscilação vertical de 4px |
| `logo-slide` | Trilha da esteira | Translação de -50% em 42s, linear e infinita; pausa no hover |

Curva padrão das transições: `cubic-bezier(.2, .7, .3, 1)` — saída rápida e chegada suave.
Durações: 180ms para estados de hover, 550–800ms para entradas.

### Revelação na rolagem

Implementada com `IntersectionObserver` e três garantias de segurança descritas em
[`architecture.md`](architecture.md#revelação-na-rolagem): o conteúdo nasce visível no CSS,
a ocultação só é ativada quando o navegador confirma suporte, e um temporizador devolve
tudo caso o observador não dispare.

---

## 6. Destaque de sintaxe

Paleta própria, dentro do mesmo sistema de cor, para os blocos que apresentam código Python.

| Classe | Cor | Token léxico |
| --- | --- | --- |
| `.doc` | `#8E8279` | Docstring |
| `.cm` | `#6E6157` | Comentário |
| `.st` | `#7FD1A0` | String |
| `.kw` | `#C77BE8` | Palavra-chave |
| `.nm` | `var(--accent-soft)` | Nome de constante à esquerda de atribuição |
| `.dn` | `#7FB7E8` | Dunder |
| `.nu` | `#7FB7E8` | Literal numérico |
| `.lc` | `#B8ABA0` | Texto padrão da linha |
| `.ln` | `#4C4238` | Numeração de linha |

O destaque é gerado em tempo de autoria, não em runtime — ver
[`architecture.md`](architecture.md#pipeline-de-autoria).
