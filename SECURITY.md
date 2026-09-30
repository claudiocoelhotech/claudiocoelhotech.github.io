# Política de segurança

## Reportando um problema

Encontrou algo que considera uma falha de segurança neste site? Escreva para
**claudiolmfcoelho@gmail.com** com o assunto `[security] claudiocoelhotech.github.io`.

Descreva o que observou, como reproduzir e o impacto que enxerga. Respondo em até 5 dias úteis.
Por favor, não abra uma issue pública antes de eu ter a chance de corrigir.

Este é um site pessoal e estático, sem programa de recompensa. O reconhecimento é público e sincero.

---

## Superfície de ataque

Vale começar pelo que **não** existe aqui, porque é o que mais reduz risco:

- **Sem servidor de aplicação.** O GitHub Pages serve arquivos estáticos. Não há código meu
  executando no servidor, então não há injeção de SQL, execução remota, deserialização insegura
  ou upload de arquivo para explorar.
- **Sem banco de dados.** Nada é armazenado, consultado ou persistido.
- **Sem autenticação.** Não existe login, sessão, cookie de sessão ou token. Não há o que roubar.
- **Sem dependências em runtime.** Nenhuma biblioteca JavaScript, nenhum CDN de terceiros,
  nenhum pacote npm. Não há cadeia de suprimentos para envenenar e não há CVE para acompanhar.
- **Sem coleta de dados.** O formulário de contato não envia nada a lugar nenhum: o `submit` é
  interceptado no navegador e exibe uma mensagem. Nenhum dado digitado sai da máquina do visitante.
- **Sem analytics, pixels ou rastreadores.**

O que sobra como superfície real: o HTML servido, o único recurso externo (Google Fonts)
e a conta do GitHub que controla o repositório.

## O que está endurecido

### Content-Security-Policy

A página declara uma CSP por `<meta http-equiv>`, com esta política:

```
default-src 'none';
base-uri   'none';
form-action 'none';
img-src    'self' data:;
font-src   https://fonts.gstatic.com;
style-src  'sha256-…' 'sha256-…' https://fonts.googleapis.com;
script-src 'sha256-…'
```

O ponto importante é o `script-src` **por hash, sem `'unsafe-inline'` e sem `'unsafe-eval'`**.
Isso significa que, se alguém conseguisse injetar uma tag `<script>` na página, o navegador se
recusaria a executá-la: só o script cujo hash está declarado roda. Os blocos de `<style>` seguem
a mesma regra.

`default-src 'none'` fecha tudo o que não foi liberado explicitamente — inclusive `connect-src`,
o que impede qualquer exfiltração por `fetch` ou `XMLHttpRequest`. `form-action 'none'` garante
que nenhum formulário, nem um injetado, consiga enviar dados para fora. `base-uri 'none'` bloqueia
o sequestro de URLs relativas por uma tag `<base>` injetada.

### Outras medidas

- **`referrer-policy: strict-origin-when-cross-origin`** — sites de destino recebem apenas a
  origem, nunca o caminho completo.
- **`rel="noopener"` em todo link externo** — a página de destino não ganha referência à janela
  de origem e não pode reescrevê-la (*tabnabbing*).
- **HTTPS obrigatório** — o GitHub Pages emite o certificado e redireciona HTTP para HTTPS.
- **Sem `eval`, sem `new Function`, sem `innerHTML` com entrada do usuário.** O único ponto em
  que texto digitado pelo visitante volta para a tela é a confirmação do formulário, e ali os
  caracteres `<`, `>` e `&` são removidos antes da inserção.
- **Sem armazenamento local.** A página não usa `localStorage`, `sessionStorage`, IndexedDB
  nem cookies. Nada persiste no navegador do visitante.

## Limitações conhecidas

O GitHub Pages **não permite definir cabeçalhos HTTP próprios**. Consequências:

| Proteção | Situação |
| --- | --- |
| `Content-Security-Policy` | Aplicada via `<meta>`, funciona |
| `Strict-Transport-Security` | Enviado pelo GitHub para o domínio `*.github.io` |
| `X-Content-Type-Options` | Enviado pelo GitHub |
| `frame-ancestors` / `X-Frame-Options` | **Não aplicável.** A diretiva `frame-ancestors` é ignorada quando a CSP vem por `<meta>`, e não há como enviar o cabeçalho. O site pode ser embutido em um `<iframe>` de terceiros. |
| `Permissions-Policy` | Não aplicável pelo mesmo motivo |

O risco prático de *clickjacking* aqui é baixo: a página não tem nenhuma ação sensível — nenhum
botão que transfira algo, autorize algo ou apague algo. O pior que um enquadramento malicioso
consegue é exibir o conteúdo fora de contexto. Se um dia o site ganhar uma ação com efeito real,
isso precisa ser reavaliado, e aí vale migrar para um host que permita cabeçalhos (Cloudflare
Pages e Netlify permitem).

## Dados pessoais expostos por escolha

O site publica **e-mail e telefone** em texto plano. Isso é intencional: é uma página de contato
profissional e o atrito de esconder o contato derrotaria o propósito dela. O custo é conhecido:
robôs de varredura vão coletar os dois e eles vão receber spam.

Se o volume incomodar, as saídas em ordem de esforço:

1. Trocar o telefone por um número secundário ou só WhatsApp Business
2. Substituir o e-mail visível por um alias descartável que encaminha para o principal
3. Montar o e-mail por JavaScript em vez de deixá-lo no HTML — atrapalha os robôs mais simples,
   mas quebra a seleção do texto e não engana raspador que executa JavaScript
4. Ligar o formulário a um serviço (Formspree, Getform, Web3Forms) e remover o contato direto

## Se o formulário passar a enviar de verdade

Hoje ele não envia nada e por isso não há obrigação de privacidade a cumprir. No dia em que for
ligado a um serviço, três coisas mudam e precisam ser tratadas:

- A CSP precisa liberar o destino em `form-action` ou `connect-src`, o que abre um caminho de
  saída de dados que hoje está fechado
- Passa a existir tratamento de dado pessoal de terceiros, com as obrigações da LGPD: informar
  a finalidade, a base legal e por quanto tempo os dados ficam guardados
- Vale um controle antiabuso (honeypot ou captcha) para o endpoint não virar relay de spam

## Conta e repositório

- **Autenticação em dois fatores ativa na conta do GitHub.** É a proteção que mais importa aqui:
  quem controla a conta controla o que é publicado no domínio.
- **Repositório público, sem segredos.** Nenhuma chave, token ou credencial foi commitada, e não
  há motivo para que isso mude — o site não fala com nenhuma API.
- **Sem GitHub Actions.** Não há workflow com permissão de escrita para ser abusado.
- Se um dia o projeto ganhar dependências, ligue o **Dependabot alerts** em Settings → Code security.

## Recalculando os hashes da CSP

Se você editar qualquer coisa dentro de `<style>` ou `<script>` no `index.html`, os hashes
declarados na CSP deixam de bater e o navegador bloqueia o bloco alterado — a página abre
sem estilo ou sem interatividade, e o console mostra um erro `Refused to apply/execute`.

Para recalcular, rode isto na pasta do projeto:

```python
import re, base64, hashlib

doc = open("index.html", encoding="utf-8").read()
sha = lambda t: "'sha256-" + base64.b64encode(hashlib.sha256(t.encode()).digest()).decode() + "'"

estilos  = re.findall(r"<style\b[^>]*>(.*?)</style>",   doc, re.S)
scripts  = re.findall(r"<script\b[^>]*>(.*?)</script>", doc, re.S)

print("style-src ",  " ".join(sha(s) for s in estilos), "https://fonts.googleapis.com")
print("script-src", " ".join(sha(s) for s in scripts))
```

Substitua as duas diretivas na tag `<meta http-equiv="Content-Security-Policy">` pelo que saiu,
recarregue a página com o cache limpo e confirme que o console está sem erros de CSP.

**Como testar rápido se está tudo certo:** abra o site, pressione F12, vá na aba Console e
recarregue. Se não houver nenhuma linha começando com `Refused to`, a política está válida.
