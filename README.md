# claudiocoelhotech.github.io

Site pessoal de **Cláudio Coelho** — Gerente de Projetos, Gestão de Contratos, Setor Público e Tecnologia.

No ar em **<https://claudiocoelhotech.github.io>**

---

## O que é

Uma página única, em três telas navegáveis sem recarregar (`_inicio`, `_sobre-mim`, `_contato`),
construída como um arquivo HTML autossuficiente e publicada pelo GitHub Pages.

A estética vem do design system que uso nos meus conteúdos no Instagram e no TikTok
([@claudiocoelho.tech](https://www.instagram.com/claudiocoelho.tech/)): fundo carvão quente,
laranja como única cor de destaque, tipografia monoespaçada nos rótulos e apresentação do
conteúdo como código Python.

## Decisões de projeto

| Decisão | Motivo |
| --- | --- |
| Arquivo único, sem build | Nada para compilar, instalar ou manter atualizado. Publicar é copiar um arquivo. |
| Sem framework e sem dependências em runtime | Menos superfície de ataque, menos peso, nenhuma quebra por atualização de terceiros. |
| CSS e JavaScript embutidos | Uma requisição HTTP para a página inteira. |
| Logos em base64 dentro do HTML | Não dependem de host externo e não quebram se algum link morrer. |
| Google Fonts como única origem externa | Único recurso de terceiros; declarado explicitamente na CSP. |
| Tema escuro fixo | O site tem uma identidade visual definida, não um tema que acompanha o sistema. |

## Estrutura

```
.
├── index.html      # o site inteiro: markup, CSS, JavaScript e imagens
├── robots.txt      # liberação para indexação + apontador do sitemap
├── sitemap.xml     # uma URL, para o Google e o Bing
├── README.md       # este arquivo
├── SECURITY.md     # política de segurança e o que está endurecido
├── LICENSE         # direitos sobre o conteúdo e as marcas de terceiros
└── .nojekyll       # desliga o processamento Jekyll no GitHub Pages
```

## Tecnologias

- HTML5 semântico
- CSS moderno: custom properties, grid, flexbox, `clamp()`, `env(safe-area-inset-*)`, marquee por `@keyframes`
- JavaScript sem bibliotecas: `IntersectionObserver` para a revelação na rolagem, `requestAnimationFrame` para a contagem dos números, `Clipboard API` para os botões de copiar
- Fontes [Archivo](https://fonts.google.com/specimen/Archivo) e [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono)
- Hospedagem: GitHub Pages

## Rodando localmente

Como não há build, basta abrir o arquivo:

```bash
# Windows
start index.html
```

Para um ambiente mais fiel ao de produção, sirva por HTTP — algumas APIs do navegador
se comportam diferente em `file://`:

```bash
python -m http.server 8000
# depois abra http://localhost:8000
```

## Onde mexer no conteúdo

Tudo vive no `index.html`. Os pontos de entrada:

| O que | Onde procurar |
| --- | --- |
| Paleta, fontes, espaçamentos | bloco `:root` no topo da `<style>` |
| Texto do hero e dos números | `<section class="screen" id="screen-inicio">` |
| Logos dos órgãos atendidos | `<div class="logo-track">` — cada `<div class="logo-chip">` |
| Resumo profissional | `<div class="pane" data-pane="perfil.py">` |
| Cases e prêmios | `<aside class="sidepanel">` da tela `_sobre-mim` |
| Ferramentas e competências | faixas `<section class="band reveal">` da tela `_sobre-mim` |
| Contatos e redes | `<aside class="contact-rail">` |

Os logos estão embutidos como `data:image/png;base64`. Para trocar um, substitua a string
base64 dentro do `<img>` correspondente. Os arquivos são recortados do fundo branco,
normalizados para o mesmo peso óptico e exibidos em escala de cinza, com a cor voltando no hover.

> **Atenção ao mexer no `index.html`:** a página declara uma Content-Security-Policy baseada em
> hashes do CSS e do JavaScript embutidos. **Qualquer alteração dentro de `<style>` ou `<script>`
> invalida o hash e o navegador passa a bloquear o bloco alterado.** Veja
> [SECURITY.md](SECURITY.md#recalculando-os-hashes-da-csp) para recalcular.

## Publicando

O GitHub Pages republica sozinho a cada push na branch `main`:

```bash
git add .
git commit -m "atualiza o site"
git push
```

A publicação leva de alguns segundos a dois minutos. O status aparece em
**Settings → Pages** e na aba **Actions** do repositório.

## Acessibilidade

- Contraste verificado: o texto secundário (`#A0948A`) atinge 6.4:1 sobre o fundo, e o laranja
  de destaque (`#EE6B1F`) atinge 6.0:1 — ambos acima do mínimo AA de 4.5:1 para texto normal
- Navegação por teclado com foco visível (`:focus-visible` com contorno laranja)
- `prefers-reduced-motion` respeitado: todas as animações são desligadas e nenhum conteúdo
  fica preso invisível esperando a rolagem
- `aria-current` na navegação, `aria-label` nos links de ícone, `aria-hidden` na cópia
  duplicada dos logos que só existe para o loop da esteira
- Sem dependência de JavaScript para ler o conteúdo: a revelação na rolagem só é ativada
  quando o navegador confirma suporte a `IntersectionObserver`, e um temporizador de segurança
  devolve tudo caso o observador não dispare

## Domínio próprio

Para servir em `claudiocoelho.tech` em vez do endereço do GitHub:

1. **Settings → Pages → Custom domain**, informe o domínio e salve
2. No registrador, crie quatro registros `A` do domínio raiz apontando para
   `185.199.108.153`, `185.199.109.153`, `185.199.110.153` e `185.199.111.153`
3. Crie um `CNAME` de `www` apontando para `claudiocoelhotech.github.io`
4. Marque **Enforce HTTPS** depois que o certificado for emitido
5. Atualize `<link rel="canonical">`, `og:url`, o `sitemap.xml` e o `robots.txt` para o novo endereço

## Licença

Veja [LICENSE](LICENSE). Resumindo: o conteúdo é meu e os logos são de seus respectivos titulares.
