# Deploy — Página Bio Thiago Alemão

Pasta do projeto: `bio-thiago-alemao/`
Arquivo principal: `index.html`
Domínio canônico: `https://thiagoalemao.com.br/`

---

## 1. Arquivos a publicar

| Caminho na hospedagem | Arquivo local | Obrigatório |
|---|---|---|
| `/index.html` | `bio-thiago-alemao/index.html` | Sim |
| `/diagnostico-financeiro-vet/` (pasta inteira) | `../diagnostico-financeiro-vet/` (projeto separado, na raiz de `c:/Programação/`) | Sim — botão "Diagnóstico" da bio aponta pra cá |
| Demais assets | Nenhum externo — `index.html` já tem CSS, JS e imagens (base64) embutidos | — |

**Estrutura final no servidor:**

```
/                                  ← raiz do domínio
├── index.html                     ← veio de bio-thiago-alemao/index.html
└── diagnostico-financeiro-vet/    ← projeto separado, com build próprio
    └── index.html                 ← link relativo da bio aponta aqui
```

**Atenção:** a bio usa link relativo `diagnostico-financeiro-vet/index.html`. Se a pasta não existir no servidor (mesmo domínio), o botão principal quebra (404). O projeto do diagnóstico tem seu próprio guia de deploy em `../diagnostico-financeiro-vet/DOCUMENTACAO_DEPLOY_FINAL.md`.

---

## 2. Configurações do servidor

- **HTTPS obrigatório** (Meta Pixel e `wa.me` exigem)
- **Content-Type** correto: `text/html; charset=utf-8`
- **Compressão**: ativar gzip/brotli (o HTML tem ~321 KB, comprime bem)
- **Cache**: `Cache-Control: public, max-age=3600` no HTML; assets imutáveis se houver
- **Redirect**: `www.thiagoalemao.com.br` → `thiagoalemao.com.br` (ou vice-versa, escolher um)
- **robots.txt** e **sitemap.xml**: opcional mas recomendado pra SEO

---

## 3. DNS

- `A` ou `CNAME` apontando o domínio para o host (Vercel, Netlify, Hostinger, etc.)
- Certificado SSL (Let's Encrypt automático na maioria dos hosts modernos)

---

## 4. Integrações já configuradas no HTML

| Item | Valor atual | Status |
|---|---|---|
| Meta Pixel ID | `678415494826119` | ✅ Ativo (PageView, InitiateCheckout, Lead) |
| WhatsApp flutuante | `+55 79 98172-6045` | ✅ Preenchido |
| Telefone JSON-LD (SEO) | `+55-79-98172-6045` | ✅ Preenchido |
| Instagram footer | `@thiagoalemaovet` | ✅ |
| E-mail contato | `contato@thiagoalemao.com.br` | ✅ — confirmar se caixa está ativa |
| Checkout Kit (Ticto) | `https://payment.ticto.app/O59BC515A` | ✅ — confirmar com Ticto que o link está ativo |
| CNPJ no rodapé | `39.504.037/0001-92` | ✅ |

---

## 5. Pendência crítica — CRM (antes do go-live)

O formulário de qualificação **ainda não envia leads para nenhum lugar**. Hoje o submit só mostra a recomendação na tela e dispara evento pro Meta Pixel.

### O que o dev do CRM precisa entregar

1. **URL do endpoint** (HTTPS) que recebe POST. Exemplo: `https://crm.cliente.com/api/leads`
2. **Autenticação**: como autenticar? (header `Authorization: Bearer ...`, token em query string, sem auth com CORS restrito, etc.)
3. **CORS**: liberar `Access-Control-Allow-Origin: https://thiagoalemao.com.br` (ou `*`) e métodos `POST, OPTIONS`
4. **Formato do payload aceito**: confirmar se o JSON abaixo serve ou se precisa mapear campos

### Payload que o formulário envia hoje (JSON, POST)

```json
{
  "nome": "string — nome informado",
  "whatsapp": "string — WhatsApp com DDD digitado pelo usuário",
  "faturamento": "ate-30k | 30-50k | 50-100k | 100k-mais | autonomo",
  "gargalo": "financeiro | precificacao | equipe | operacional | crescimento",
  "relato": "string — texto livre, opcional, até 600 chars",
  "ts": "2026-05-27T12:34:56.000Z — timestamp ISO 8601 UTC"
}
```

Sugiro adicionar também `source: "bio-instagram"` para o CRM saber a origem.

### Local da configuração no HTML

Linha **2361**, constante `LEAD_ENDPOINT`. Quando o endpoint estiver pronto, basta:

```js
const LEAD_ENDPOINT = 'https://crm.cliente.com/api/leads';
```

Se precisar de header de auth, é uma alteração pequena no bloco `fetch` logo abaixo (linhas 2415-2425).

---

## 6. Checklist de QA pré-deploy

- [ ] Abrir a página em mobile real (não só DevTools) — testar sticky CTA e WhatsApp flutuante
- [ ] Clicar nos 2 cards de oferta e confirmar destino correto
- [ ] Preencher quiz com cada faixa de faturamento e verificar se mostra a recomendação certa:
  - Até R$30k / Autônomo → Kit
  - R$30-50k / R$50-100k → Diagnóstico
  - Acima R$100k → Plano sob medida
- [ ] Abrir todos os 7 itens do FAQ
- [ ] Validar links do rodapé (Instagram, e-mail)
- [ ] Clicar no WhatsApp flutuante e confirmar que abre conversa com mensagem pré-preenchida
- [ ] Meta Pixel: instalar [Meta Pixel Helper](https://chrome.google.com/webstore/detail/meta-pixel-helper/fdgfkebogiimcoedlicjlajpkdmockpc) e confirmar PageView ao carregar
- [ ] Lighthouse mobile: meta de 90+ em Performance / Acessibilidade / SEO
- [ ] Testar em iOS Safari e Android Chrome

---

## 7. Pós-deploy

- [ ] Cadastrar a URL no Meta Events Manager (verificar domínio para o Pixel)
- [ ] Submeter sitemap ao Google Search Console
- [ ] Compartilhar a URL no Facebook Sharing Debugger ([developers.facebook.com/tools/debug](https://developers.facebook.com/tools/debug/)) pra forçar refresh do Open Graph
- [ ] Quando CRM estiver plugado, fazer 1 lead de teste e confirmar que chegou
- [ ] Verificar evento "Lead" no Meta Events Manager após primeiro envio real

---

## 8. Resumo do que está faltando

| # | Item | Bloqueia deploy? |
|---|---|---|
| 1 | Subir pasta `diagnostico-financeiro-vet/` junto | Sim |
| 2 | URL + auth + CORS do CRM | Não bloqueia, mas leads serão perdidos até plugar |
| 3 | Confirmar e-mail `contato@thiagoalemao.com.br` ativo | Não, mas e-mails vão sumir |
| 4 | Confirmar link Ticto ativo | Sim — sem isso ninguém compra o Kit |
