# Arquitetura

## Modelo

Aplicação de página única, sem servidor de aplicação e sem estado persistido. As três telas
coexistem no DOM desde o carregamento e a navegação alterna a visibilidade delas. Não há
requisição de rede após o documento inicial, exceto a folha de estilos do Google Fonts e os
arquivos de fonte que ela referencia.

```
GET /                    →  index.html   313 KB   (markup + CSS + JS + imagens)
GET fonts.googleapis.com →  CSS das duas famílias
GET fonts.gstatic.com    →  arquivos woff2
```

Uma requisição para a aplicação. Nenhuma para imagem: os quatorze logotipos são
`data:image/png;base64` e os dezesseis ícones são SVG inline.

## Composição do documento

```
index.html
├── <head>
│   ├── meta Content-Security-Policy      hashes SHA-256 dos blocos embutidos
│   ├── meta referrer                     strict-origin-when-cross-origin
│   ├── metadados                         title, description, canonical, Open Graph, theme-color
│   ├── link icon                         favicon SVG como data URI
│   ├── link fonts.googleapis.com         único recurso externo
│   └── <style> reset                     box-sizing, img, [hidden]
└── <body>
    ├── <style> principal                 ~23 KB, 260 regras, 15 custom properties
    ├── .shell
    │   ├── header.topbar                 fixo, com barra de progresso
    │   ├── main
    │   │   ├── section#screen-inicio      hero + faixas em rolagem
    │   │   ├── section#screen-sobre-mim   editor + painel de cases + faixas
    │   │   └── section#screen-contato     contatos + formulário
    │   └── footer.footer
    └── <script>                          ~7 KB, 177 linhas, IIFE única
```

O `<style>` principal fica no início do `<body>` e não no `<head>` por decisão de
empacotamento: mantém o `<head>` restrito a metadados e deixa o par markup+estilo contíguo.
Não há impacto de renderização, porque o navegador só pinta depois de processar a folha
de qualquer forma.

## Roteamento

Três rotas em `ORDER = ['inicio', 'sobre-mim', 'contato']`.

A função `show(id)` alterna a propriedade `hidden` das três `<section class="screen">`,
sincroniza `aria-current` na navegação, atualiza o contador do rodapé, rola para o topo e
reavalia os elementos reveláveis da tela que entrou.

O estado vai para a URL por `history.replaceState`, como fragmento simples (`#contato`).
Usa-se `replaceState` e não `pushState` de propósito: o botão "voltar" do navegador sai do
site em vez de percorrer telas, que é o comportamento esperado numa página de portfólio.
Na carga, o fragmento é lido e a rota correspondente é aberta.

Todo elemento com `data-go` navega — isso cobre tanto os botões da barra superior quanto as
chamadas para ação no fim da home, sem duplicar o manipulador.

> `[hidden]{display:none !important}` está no reset por necessidade, não por estilo.
> As seções declaram `display: flex`, e uma regra de autor vence a regra do agente do
> usuário. Sem o `!important`, a alternância de telas não funciona.

## Revelação na rolagem

O requisito é que o conteúdo apareça conforme a rolagem sem que nada fique permanentemente
invisível se o JavaScript falhar. Três camadas garantem isso:

1. **O CSS entrega tudo visível.** Não há `opacity: 0` na folha aplicado incondicionalmente.
   A ocultação vive sob o seletor `.js-reveal .reveal`, e a classe `js-reveal` só entra no
   elemento raiz depois que o script confirma `'IntersectionObserver' in window`.
2. **O observador revela ao entrar na viewport**, com `rootMargin: '0px 0px -12% 0px'` e
   `threshold: 0.12`, e deixa de observar o elemento em seguida.
3. **Um temporizador de 2400ms** revela o que já está na viewport da tela aberta, caso o
   observador não dispare. Ele ignora elementos dentro de telas ocultas, para que as faixas
   das outras rotas ainda animem quando forem abertas.

Sob `prefers-reduced-motion: reduce` a classe `js-reveal` nem chega a ser adicionada.

Dentro de um elemento revelado disparam três efeitos: contagem dos números (`countUp`),
datilografia dos rótulos (`typeIn`) e as transições escalonadas dos filhos de `.stagger`,
resolvidas só por CSS com `transition-delay` por `nth-child`.

## Pipeline de autoria

O repositório não tem build, mas o `index.html` é gerado. Os scripts de autoria vivem fora
do repositório e produzem três coisas:

**Normalização dos logotipos.** Cada arquivo de origem passa por remoção de fundo por
preenchimento a partir das bordas — só o claro conectado à moldura vira transparente, o que
preserva branco interno como as letras da Caixa e da PRF —, recorte pela caixa delimitadora
do canal alfa, redimensionamento por área alvo para igualar o peso óptico entre marcas de
proporções muito diferentes, centralização em canvas de 240×132 e codificação em base64.
O resultado é que todos os ladrilhos têm o mesmo canvas e o logotipo cai exatamente no
centro, sem ajuste manual por marca.

**Destaque de sintaxe.** Um tokenizador de Python reduzido percorre o fonte linha a linha,
com estado para docstrings, e emite `<span class="ln">` e `<span class="lc">` por linha
lógica. A numeração fica no topo da linha quando o texto quebra, porque cada linha lógica é
uma linha de grade, e não uma quebra visual.

**Montagem e CSP.** O esqueleto HTML, os metadados e o favicon são adicionados ao corpo;
em seguida os hashes SHA-256 de cada `<style>` e `<script>` do documento já montado são
calculados e injetados na diretiva. Duas asserções falham a montagem se aparecer atributo
`style=` inline ou `<script src=>`, porque qualquer um dos dois quebraria a política por hash.

## Acoplamento entre CSP e conteúdo

A política declara o hash do conteúdo exato de cada bloco embutido:

```
script-src 'sha256-<hash do único <script>>'
style-src  'sha256-<reset>' 'sha256-<folha principal>' https://fonts.googleapis.com
```

Isso torna impossível executar script injetado, e ao mesmo tempo cria um acoplamento real:
**um espaço a mais dentro de `<style>` ou `<script>` invalida o hash** e o navegador recusa
o bloco. O sintoma é a página abrir sem estilo ou sem interatividade, com
`Refused to apply/execute` no console.

Duas alternativas foram consideradas e descartadas: `'unsafe-inline'` anula o benefício, e
`nonce` exige gerar valor por requisição, o que um host estático não faz.

### Recalculando os hashes

Rode na pasta do projeto depois de qualquer edição dentro de `<style>` ou `<script>`:

```python
import re, base64, hashlib

doc = open("index.html", encoding="utf-8").read()
sha = lambda t: "'sha256-" + base64.b64encode(hashlib.sha256(t.encode()).digest()).decode() + "'"

estilos = re.findall(r"<style\b[^>]*>(.*?)</style>",   doc, re.S)
scripts = re.findall(r"<script\b[^>]*>(.*?)</script>", doc, re.S)

print("style-src ", " ".join(sha(s) for s in estilos), "https://fonts.googleapis.com")
print("script-src", " ".join(sha(s) for s in scripts))
```

Substitua as duas diretivas na tag `<meta http-equiv="Content-Security-Policy">` pelo que
saiu e recarregue com o cache limpo.

**Verificação:** abra o console do navegador (F12) e recarregue. Se não houver nenhuma linha
começando com `Refused to`, a política está válida.

## Estado

Não há estado persistido em lugar nenhum. Sem `localStorage`, sem `sessionStorage`, sem
IndexedDB, sem cookie. O único estado é em memória: qual rota está aberta e quais elementos
já foram revelados. Recarregar a página reinicia tudo, exceto a rota, que vem do fragmento
da URL.

O formulário de contato não tem destino. O `submit` é interceptado com `preventDefault()`
e a resposta é uma mensagem na própria página, que informa a natureza do protótipo e exibe
o e-mail. Nenhum dado digitado sai do navegador.

## Decisões e seus custos

| Decisão | Ganho | Custo aceito |
| --- | --- | --- |
| Arquivo único sem build | Nada para instalar, compilar ou manter atualizado; publicar é copiar um arquivo | Edições são feitas num arquivo de 313 KB, sem separação por módulo |
| Imagens em base64 | Zero requisição de imagem, nenhum link externo para morrer | Cresce o documento em ~130 KB e impede cache separado das imagens |
| CSS e JS embutidos | Uma requisição para a aplicação inteira | O navegador não pode cachear estilo e script independentemente do conteúdo |
| CSP por hash | Bloqueia execução de script injetado | Qualquer edição nos blocos exige recálculo |
| Sem framework | Sem cadeia de dependências, sem CVE para acompanhar, sem migração de versão maior | Roteamento, revelação e animação são escritos à mão |
| Sintaxe ES5 | Roda sem transpilação em qualquer navegador que tenha as APIs de DOM usadas | Código mais verboso que o equivalente moderno |
| Tema único escuro | Metade das combinações de cor para projetar e testar | Não acompanha a preferência de tema do sistema |
