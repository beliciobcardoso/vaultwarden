# INF-07 — HTTPS local via ngrok (PoC temporária)

**Status:** done · **Prioridade:** P0 · **Backlog:** [INF-07](../backlog.md)

## Contexto
Cliente Bitwarden Web recusa fluxos SSO quando `DOMAIN` é `http://` (erro "Insecure URL not allowed. All URLs must use HTTPS."). Bloqueia INF-05, POC-01..08 inteiros.

## Problema
PoC roda em `http://localhost:8080`. Sem HTTPS, o cliente web bloqueia o redirect SSO antes mesmo de chamar o Azure AD.

## Objetivo
Expor o Vaultwarden local via URL HTTPS estável durante a PoC. Solução **temporária** — em produção usa domínio próprio + cert público (INF futuro).

## Escopo
**Dentro:**
- Usar `ngrok` (já instalado + autenticado em `~/.config/ngrok/ngrok.yml`) para expor `http://localhost:8080` em HTTPS
- Reservar domínio estático ngrok (se plano permitir) — evita re-config do Azure AD a cada sessão
- Atualizar `DOMAIN` no `.env` local para a URL ngrok HTTPS
- Adicionar a URL ngrok como redirect URI adicional na App Registration Azure AD (INF-04)
- Documentar comando de subida do túnel + como parar

**Fora:**
- HTTPS em produção (domínio próprio, cert Let's Encrypt, reverse proxy) — vira PRD dedicado quando prod entrar em pauta
- Caddy/nginx local — abandonado em favor de ngrok pela natureza temporária da PoC

## Critérios de aceitação
- [x] Túnel ngrok up expondo `localhost:8080` em HTTPS
- [x] URL ngrok documentada no `.env` local (não versionada)
- [x] Redirect URI ngrok cadastrada no Azure AD App Registration (INF-04)
- [x] `DOMAIN` no `.env` = URL ngrok HTTPS
- [x] Vaultwarden reinicia e responde no `<ngrok-url>/alive`
- [x] Cliente web abre sem erro "Insecure URL"
- [x] Comando de subida documentado neste PRD

## Comandos

```bash
# Subir tunel (foreground; deixa rodando durante a sessao)
ngrok http 8080

# Ou em background com dominio estatico reservado (recomendado)
ngrok http --url=<seu-dominio-reservado>.ngrok-free.app 8080

# Ver URL ativa
curl -s http://localhost:4040/api/tunnels | jq -r '.tunnels[0].public_url'
```

## Dependências
- INF-04 (App Registration existe — vai receber redirect URI adicional)
- Bloqueia: INF-05, POC-01..08

## Riscos & mitigações
- **URL ngrok muda a cada restart (free tier)** → reservar domínio estático ou aceitar re-cadastrar redirect URI a cada sessão.
- **Túnel expõe PoC pra internet** → mitigar com `ngrok http --basic-auth` ou domínio obscuro; nunca subir com dados reais.
- **Client secret Azure AD tem validade 90d** → alerta manual (herdado de INF-04).

## Referências
- INF-04 (App Registration Azure AD)
- INF-05 (SSO config — bloqueada por este)
- ngrok docs: https://ngrok.com/docs
