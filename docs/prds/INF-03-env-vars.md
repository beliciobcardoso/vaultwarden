# INF-03 — Configurar variáveis de ambiente (`.env`)

**Status:** todo · **Prioridade:** P0 · **Backlog:** [INF-03](../backlog.md)

## Contexto
Vaultwarden é 100% configurado via env vars (ver `src/config.rs` e `.env.template` na raiz — ~88KB de opções). PoC precisa de `.env` local com valores da Nova Corrente, sem versionar segredos.

## Problema
Sem `.env` estruturado, cada dev configura de forma inconsistente e há risco real de vazar `ADMIN_TOKEN`, `SMTP_PASSWORD`, `SSO_CLIENT_SECRET` no git.

## Objetivo
Um `.env.example` versionado (chaves + valores fictícios/placeholders) + `.env` local (real, git-ignored) cobrindo tudo que a PoC precisa: DB, SMTP, admin, SSO Azure AD, domínio, MFA.

## Escopo
**Dentro:**
- `.env.example` na raiz com todas as chaves necessárias à PoC, com comentários e defaults seguros
- `.env` real (não versionado) — o `.gitignore` do repo já cobre `.env`, confirmar
- Grupos de variáveis: `DATABASE_URL`, `ADMIN_TOKEN`, `DOMAIN`, `SIGNUPS_ALLOWED=false`, `SMTP_*`, `SSO_ENABLED=true` + `SSO_*` (Azure AD), `PUSH_*` (se aplicável)
- Documentar geração de `ADMIN_TOKEN` seguro (`openssl rand -base64 48` ou `argon2` conforme docs Vaultwarden)

**Fora:** vault de segredos (Vault/Doppler/1Password) — PoC roda com `.env` local mesmo.

## Critérios de aceitação
- [ ] `.env.example` versionado, sem segredos reais
- [ ] `.env` local existe, funcional, ignorado pelo git (`git status` não mostra)
- [ ] Todas as vars do INF-02 e INF-05 documentadas em `.env.example`
- [ ] `SIGNUPS_ALLOWED=false` explícito (evita cadastro fora do domínio corporativo)
- [ ] Comentário no `.env.example` explica como gerar `ADMIN_TOKEN` seguro
- [ ] Vaultwarden sobe consumindo `.env` (via `env_file` no compose)

## Dependências
- INF-02 (compose consome o `.env`)
- INF-04 fornece valores reais de `SSO_CLIENT_ID`/`SSO_CLIENT_SECRET`/`SSO_AUTHORITY`

## Riscos & mitigações
- **Vazamento de segredo em commit** → confirmar `.gitignore`; pre-commit hook (`gitleaks`) opcional.
- **Chave nova no upstream sem documentar** → revisar `.env.template` do upstream a cada merge.

## Referências
- `.env.template` (raiz — referência upstream)
- `src/config.rs`
