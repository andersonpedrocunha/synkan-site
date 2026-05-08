# Synkan Site · Cloudflare Pages Deploy

Pacote pronto pra subir o site público (`synkan.com.br`) na Cloudflare Pages.

## Estrutura

```
site-deploy/
├── index.html              ← landing
├── upgrade.html            ← /upgrade — página de planos com checkout Stripe
├── termos.html             ← /termos — Termos de Uso
├── privacidade.html        ← /privacidade — Política LGPD
├── docs.html               ← /docs — documentação
├── status.html             ← /status — saúde dos sistemas
├── changelog.html          ← /changelog — histórico de versões
├── 404.html                ← servida automaticamente em qualquer 404
├── _redirects              ← rotas amigáveis e compat de URLs antigas
├── _headers                ← segurança (CSP, HSTS, X-Frame, etc) + cache
├── robots.txt              ← regras pra crawlers
├── sitemap.xml             ← lista de URLs pro Google indexar
└── .gitignore              ← ignora arquivos de backup
```

## Subir o site (passo a passo, ~30 minutos)

### 1. GitHub

```bash
cd site-deploy
git init
git add .
git commit -m "Initial commit: Synkan public site"
# Crie um repo vazio em github.com chamado "synkan-site" (ou outro nome)
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/synkan-site.git
git push -u origin main
```

### 2. Cloudflare Pages

1. Vai em [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. Autoriza o GitHub e seleciona o repositório `synkan-site`
3. Configurações de build:
   - **Production branch:** `main`
   - **Framework preset:** None
   - **Build command:** *(deixe vazio)*
   - **Build output directory:** `/`
4. Clica **Save and Deploy**. Em ~1 minuto o site tá no ar em `synkan-site.pages.dev`

### 3. Conectar o domínio synkan.com.br

Antes disso, você precisa ter o domínio na Cloudflare:

1. Cloudflare → **Add site** → digita `synkan.com.br` → escolhe plano **Free**
2. A Cloudflare te mostra os nameservers (tipo `gabe.ns.cloudflare.com` e `lara.ns.cloudflare.com`)
3. No painel do **registro.br**, troca os DNS pelos da Cloudflare (propaga em 4-24h)

Depois, no Pages:

1. Pages → seu projeto → **Custom domains** → **Set up a custom domain**
2. Digita `synkan.com.br` → confirma
3. Repete pra `www.synkan.com.br`
4. Cloudflare cria os registros DNS automaticamente

Em alguns minutos: https://synkan.com.br no ar com SSL.

### 4. Email Routing (15 minutos)

Cloudflare → seu domínio → **Email** → **Email Routing** → **Get started**

Cria os endereços (todos encaminham pro seu Gmail pessoal):

```
contato@synkan.com.br      →  seu-gmail@gmail.com
suporte@synkan.com.br      →  seu-gmail@gmail.com
dpo@synkan.com.br          →  seu-gmail@gmail.com
cobranca@synkan.com.br     →  seu-gmail@gmail.com
alertas@synkan.com.br      →  seu-gmail@gmail.com  (também atende inscrição em alertas de status)
```

A Cloudflare confirma seu email pessoal e cria os MX records sozinha. **Grátis e ilimitado.**

### 5. Verificação final

Abre no navegador e confirma:

- [ ] https://synkan.com.br carrega (landing)
- [ ] https://synkan.com.br/termos abre
- [ ] https://synkan.com.br/privacidade abre
- [ ] https://synkan.com.br/upgrade abre
- [ ] https://synkan.com.br/docs abre
- [ ] https://synkan.com.br/changelog abre
- [ ] https://synkan.com.br/status abre
- [ ] https://synkan.com.br/url-aleatoria-que-nao-existe → mostra a 404 customizada
- [ ] Manda email pra contato@synkan.com.br → chega no Gmail
- [ ] Cadeado verde no navegador (SSL OK)

## Pra atualizar o site no futuro

```bash
# edita o que precisa
git add .
git commit -m "Atualiza copy da landing"
git push
```

A Cloudflare Pages faz deploy automático em ~1 minuto.

## Subdomínios futuros

Quando você for subir o app, billing e API:

| Subdomínio | Onde hospedar | Como conectar |
|---|---|---|
| `app.synkan.com.br` | Outro Cloudflare Pages (kanban_v8.html como index.html) | Pages → Custom domain → app.synkan.com.br |
| `billing.synkan.com.br` | Fly.io (rodando billing/billing_server.py) | DNS CNAME → seu-app.fly.dev |
| `api.synkan.com.br` | Fly.io (rodando kanban_sync_server.py) | DNS CNAME → seu-api.fly.dev |
| `status.synkan.com.br` | Better Stack / Instatus (opcional) | DNS CNAME pro provider |

A landing já aponta automaticamente pra `app.synkan.com.br` (botão "Entrar →") e o app já detecta o subdomínio pra usar `api.synkan.com.br` e `billing.synkan.com.br` em produção.

## Quando algo der errado

- **Site mostra 404 na home** — confirma que `index.html` está na raiz do repo
- **Links internos quebram** — confirma que os arquivos foram copiados (não só renomeados) e que `_redirects` está na raiz
- **404 customizada não aparece** — confirma que o arquivo se chama exatamente `404.html` (Cloudflare exige esse nome)
- **Email não chega** — espera 1h pra propagar, depois verifica em Email Routing → Routing rules
- **DNS não propaga** — abre [whatsmydns.net](https://whatsmydns.net) e cola synkan.com.br pra ver onde já chegou
- **CSP bloqueia algum script** — abre o console do navegador, identifica o domínio bloqueado, adiciona em `_headers` na linha `Content-Security-Policy`

## Arquivos `.bak`

Se você ver `*.html.bak` no diretório, pode deletar. São backups que o `sed` cria. O `.gitignore` já ignora.
