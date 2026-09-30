# Stack técnica

## Linguagens

| Linguagem | Versão / nível | Onde |
| --- | --- | --- |
| HTML | HTML5 | Markup semântico, atributos ARIA, `data-*` para vínculo com o comportamento |
| CSS | CSS3, sem pré-processador | Custom properties, Grid, Flexbox, `clamp()`, `env()`, `@keyframes`, `mask`/gradiente |
| JavaScript | **Sintaxe ES5**, APIs de DOM modernas | Uma IIFE de 177 linhas, 7 KB |

A escolha de escrever a sintaxe em ES5 — `var`, expressões de função, nenhuma arrow function,
nenhum template literal — é deliberada. Sem transpilação, o limite de compatibilidade passa a
ser determinado pelas **APIs** que a página usa, não pela sintaxe que o parser precisa aceitar.
Isso torna a matriz de suporte previsível e verificável item a item.

## APIs do navegador em uso

| API | Uso | Degradação quando ausente |
| --- | --- | --- |
| `IntersectionObserver` | Revelação de faixas na rolagem | Testada com `in window`; sem ela, todo o conteúdo permanece visível |
| `requestAnimationFrame` | Contagem animada dos números | Só é chamada dentro do caminho de revelação |
| `navigator.clipboard.writeText` | Botões de copiar e-mail e telefone | Rejeição tratada; o botão passa a pedir seleção manual |
| `window.matchMedia` | Detecção de `prefers-reduced-motion` | Verificada antes do uso; ausência equivale a movimento permitido |
| `history.replaceState` | Fragmento da rota na URL | Envolvida em `try/catch` |
| `Element.scrollIntoView` | Chamada de rolagem do hero | Comportamento `smooth` ou `auto` conforme a preferência de movimento |
| `Element.closest` | Descobrir a tela dona de um elemento revelável | — |
| `NodeList.prototype.forEach` | Iteração sobre seleções | — |
| `classList`, `dataset` | Estado visual e vínculo de comportamento | — |

**Não usadas, por decisão:** `localStorage`, `sessionStorage`, IndexedDB, cookies, `fetch`,
`XMLHttpRequest`, WebSocket, Service Worker, `eval`, `new Function`, `innerHTML` com entrada
do usuário.

## Tipografia

| Família | Pesos | Origem | Licença |
| --- | --- | --- | --- |
| [Archivo](https://fonts.google.com/specimen/Archivo) | 400, 500, 600, 700, 800 | Google Fonts | SIL OFL 1.1 |
| [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) | 400, 500, 700 | Google Fonts | SIL OFL 1.1 |

Carregadas com `display=swap` e precedidas de `preconnect` para `fonts.googleapis.com` e
`fonts.gstatic.com`. Cada token de fonte declara pilha de fallback completa, de modo que a
página permanece legível e com métricas próximas se a requisição falhar.

Este é o **único recurso de terceiros** do projeto e está declarado explicitamente na CSP
(`style-src https://fonts.googleapis.com`, `font-src https://fonts.gstatic.com`).

## Assets

| Tipo | Quantidade | Formato | Entrega |
| --- | --- | --- | --- |
| Logotipos de organizações | 14 (7 únicos, duplicados para o loop) | PNG 240×132, fundo transparente | `data:image/png;base64` embutido |
| Ícones de interface | 16 | SVG com `stroke="currentColor"` | Inline no markup |
| Favicon | 1 | SVG | `data:image/svg+xml` no `<link rel="icon">` |

Nenhuma requisição de imagem. Os ícones herdam a cor do contexto por `currentColor`,
o que dispensa variantes por estado.

## Hospedagem e entrega

| | |
| --- | --- |
| Host | GitHub Pages |
| Origem | Branch `main`, diretório raiz |
| Pipeline | `pages-build-deployment` padrão do GitHub, sem workflow customizado |
| Jekyll | Desligado por `.nojekyll` |
| TLS | Certificado e redirecionamento HTTP→HTTPS providos pelo GitHub |
| Compressão | Brotli/gzip negociados pelo CDN do GitHub |
| Latência de publicação | 20 segundos a 2 minutos após o push |

O documento tem 313 KB não comprimido. Como é quase todo texto — markup, CSS, JavaScript e
base64 —, comprime bem no transporte.

### Domínio próprio

Para servir sob um domínio próprio em vez do endereço `*.github.io`:

1. **Settings → Pages → Custom domain**, informe o domínio e salve — isso cria um arquivo
   `CNAME` na raiz do repositório
2. No registrador, aponte o domínio raiz para os quatro IPs do GitHub Pages:
   `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` (registros `A`)
3. Crie um `CNAME` de `www` apontando para `claudiocoelhotech.github.io`
4. Marque **Enforce HTTPS** depois que o certificado Let's Encrypt for emitido
5. Atualize no `index.html` o `<link rel="canonical">` e a tag `og:url`, e também o
   `sitemap.xml` e o `robots.txt` — os três apontam para o endereço antigo

O passo 5 mexe apenas no `<head>`, fora dos blocos `<style>` e `<script>`, portanto **não**
exige recálculo dos hashes da CSP.

## Ferramental de autoria

Ferramentas usadas para **gerar** o `index.html`. Não fazem parte do runtime nem estão no
repositório; quem só edita conteúdo não precisa de nenhuma delas.

| Ferramenta | Papel |
| --- | --- |
| Python 3 | Montagem do documento, cálculo dos hashes da CSP, tokenizador de destaque de sintaxe |
| Pillow | Normalização dos logotipos: remoção de fundo, recorte, redimensionamento por área e centralização |
| NumPy | Preenchimento a partir das bordas na remoção de fundo |
| Playwright + Chromium | Verificação de regressão visual, checagem de overflow horizontal e leitura do console em busca de violação de CSP |

## Compatibilidade

Determinada pela API mais recente exigida, que é `IntersectionObserver`.

| Navegador | Mínimo | Observação |
| --- | --- | --- |
| Chrome / Edge | 58 | |
| Firefox | 55 | |
| Safari | 12.1 | |
| Safari iOS | 12.2 | `env(safe-area-inset-*)` a partir do 11.2 |
| Chrome Android | 58 | |

Abaixo desses patamares a página continua legível e navegável: o conteúdo permanece visível
porque a ocultação depende da detecção de suporte, e a troca de telas usa apenas `hidden` e
manipuladores de clique.

**Sem suporte a Internet Explorer.** Custom properties, Grid e `IntersectionObserver` não
existem nele, e não há polyfill no projeto.

## Dependências

Nenhuma. Sem `package.json`, sem `node_modules`, sem lockfile, sem bundler, sem
pré-processador de CSS, sem polyfill.

A consequência prática é que não existe cadeia de suprimentos para envenenar, não há CVE de
dependência para acompanhar e não há atualização de versão maior para migrar. O site tende a
continuar funcionando sem manutenção enquanto os padrões da web mantiverem
retrocompatibilidade.
