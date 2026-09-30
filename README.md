# claudiocoelhotech.github.io

Site pessoal de Cláudio Coelho, publicado em <https://claudiocoelhotech.github.io>.

Aplicação de página única, sem build e sem dependências em runtime, entregue como um
documento HTML autossuficiente e hospedada no GitHub Pages.

---

## Visão geral técnica

| | |
| --- | --- |
| **Tipo** | Single Page Application estática, três rotas, sem recarga de página |
| **Linguagens** | HTML5, CSS3, JavaScript (sintaxe ES5, APIs de DOM modernas) |
| **Frameworks** | Nenhum |
| **Dependências em runtime** | Nenhuma além de duas famílias tipográficas do Google Fonts |
| **Processo de build** | Nenhum no repositório — o artefato publicado é o próprio fonte |
| **Artefato** | `index.html`, 313 KB, uma única requisição para a aplicação inteira |
| **Hospedagem** | GitHub Pages, branch `main`, diretório raiz |
| **TLS** | Certificado e redirecionamento HTTP→HTTPS providos pelo GitHub |
| **Content Security Policy** | Hashes SHA-256 dos blocos embutidos, sem `unsafe-inline` e sem `unsafe-eval` |

## Documentação

| Documento | Conteúdo |
| --- | --- |
| [`docs/architecture.md`](docs/architecture.md) | Composição do documento, modelo de renderização, roteamento, sistema de revelação na rolagem, pipeline de assets e o acoplamento entre a CSP e o conteúdo |
| [`docs/design-system.md`](docs/design-system.md) | Tokens de cor, escala tipográfica, espaçamento, inventário de componentes, motion e razões de contraste medidas |
| [`docs/tech-stack.md`](docs/tech-stack.md) | Linguagens, APIs do navegador em uso, tipografia, hospedagem, ferramental de autoria e matriz de compatibilidade |
| [`docs/conventions.md`](docs/conventions.md) | Convenções de nomenclatura de CSS e JavaScript, organização da folha de estilos e padrão de commits |
| [`LICENSE`](LICENSE) | Direitos sobre conteúdo, código e marcas de terceiros |

## Estrutura do repositório

```
.
├── index.html              artefato publicado: markup, CSS, JS e imagens
├── robots.txt              diretivas de indexação e apontador do sitemap
├── sitemap.xml             uma URL canônica
├── .nojekyll               desliga o pipeline Jekyll do GitHub Pages
├── .gitignore
├── README.md
├── LICENSE
└── docs/
    ├── architecture.md
    ├── design-system.md
    ├── tech-stack.md
    └── conventions.md
```

## Ambiente local

Não há instalação, transpilação ou bundling. Abrir o arquivo já é suficiente para a maior
parte do trabalho:

```bash
start index.html          # Windows
```

Para um ambiente fiel ao de produção — origem HTTP, mesma resolução de caminhos relativos
e mesmo comportamento de `fetch` e de política de origem:

```bash
python -m http.server 8000
# http://localhost:8000
```

## Publicação

O GitHub Pages reconstrói a cada push em `main`. Não há workflow customizado no repositório;
o deploy é o `pages-build-deployment` padrão do GitHub.

```bash
git add .
git commit -m "..."
git push
```

Latência típica entre o push e a página servida: de 20 segundos a 2 minutos. O status fica em
**Settings → Pages** e na aba **Actions**.

## Restrição operacional

> A CSP declara hashes SHA-256 dos blocos `<style>` e `<script>` embutidos. Editar qualquer
> um desses blocos invalida o hash correspondente e o navegador passa a recusar o bloco.
> O acoplamento e o procedimento de recálculo estão em
> [`docs/architecture.md`](docs/architecture.md#acoplamento-entre-csp-e-conteúdo).
