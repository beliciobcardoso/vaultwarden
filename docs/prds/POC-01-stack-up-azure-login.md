# POC-01 — Subir stack local e autenticar via Azure AD (SSO padrão)

**Status:** done · **Prioridade:** P0 · **Backlog:** [POC-01](../backlog.md)

## Contexto
Primeiro teste ponta-a-ponta do PoC: subir stack local (INF-02..05) e completar um login SSO via Azure AD com um usuário real do tenant de teste.

## Problema
Validar que INF-02..INF-05 estão corretos, integrados e produzem um login funcional. Sem esse marco, todos os POCs seguintes ficam bloqueados.

## Objetivo
Login SSO completo em ambiente local, sessão ativa no web vault, sem erros no log.

## Escopo
**Dentro:**
- Subir stack via `docker compose up -d`
- Abrir web vault local
- Clicar em "Log in with SSO", completar fluxo com usuário Azure AD de teste
- Confirmar sessão criada e vault vazio acessível
- Coletar log de sucesso

**Fora:**
- Restrição de domínio (POC-02)
- MFA step-up (POC-03)
- Validação de criptografia (POC-04)

## Critérios de aceitação
- [x] Stack sobe sem erros (`docker compose ps` → 3 serviços healthy)
- [x] Web vault acessível via HTTPS (túnel ngrok — INF-07; `http://localhost:8080` bloqueado pelo cliente Bitwarden Web)
- [x] Fluxo SSO completa com usuário `belicio.cardoso@novacorrente.ind.br`
- [x] Sessão persiste após refresh (log confirma múltiplos `/api/accounts/profile` 200 OK + WebSocket ativo)
- [x] Log Vaultwarden sem `ERROR`/`WARN` no fluxo SSO
- [x] Roteiro reproduzível documentado (ver seção "Roteiro" abaixo)

## Dependências
INF-02, INF-03, INF-04, INF-05

## Riscos & mitigações
- **Falha por config errada** → checar INF-05 primeiro (variáveis SSO), depois logs.
- **Redirect loop mesmo em SSO padrão** → não deveria (BUG-01 é só `SSO_ONLY`), mas se ocorrer, tratar como novo bug e registrar no backlog.

## Roteiro (validado 2026-09-07)

1. `docker compose up -d` (usa `newchainsafe/vaultwarden:1.34.3-nc.1` — build local INF-06)
2. Subir túnel HTTPS: `ngrok http 8080` (INF-07)
3. Confere redirect URI no Azure AD App Registration bate com `<url-ngrok>/identity/connect/oidc-signin`
4. `.env`: `DOMAIN=<url-ngrok>`, `SSO_ALLOW_UNKNOWN_EMAIL_VERIFICATION=true` (Azure AD não envia claim)
5. `docker compose up -d --force-recreate vaultwarden` (restart não relê `env_file`)
6. Abrir URL ngrok → digitar email `@novacorrente.ind.br` → "Usar autenticação única"
7. Redirect Microsoft → login → consent (1ª vez) → callback
8. Vault vazio acessível, sessão persiste em refresh

## Referências
- INF-02..INF-05, INF-07
