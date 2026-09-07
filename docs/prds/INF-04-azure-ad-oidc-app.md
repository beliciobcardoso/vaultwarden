# INF-04 — Registrar aplicação OIDC no Azure AD (tenant de teste)

**Status:** done · **Prioridade:** P0 · **Backlog:** [INF-04](../backlog.md)

## Contexto
SSO Nova Corrente = Azure AD. Antes de configurar o Vaultwarden (INF-05), a aplicação OIDC precisa existir no tenant.

## Problema
Sem App Registration no Azure AD com redirect URIs corretos, o Vaultwarden não consegue completar o fluxo OIDC.

## Objetivo
Uma App Registration no tenant de **teste** (não produção) com Client ID, Client Secret, redirect URIs e claims configurados de forma a suportar POC-01..POC-04.

## Escopo
**Dentro:**
- App Registration no tenant de teste (nome sugerido: `NewChainSafe PoC`)
- Client Secret gerado, validade curta (ex.: 90 dias) — armazenado apenas no `.env` local
- Redirect URI: `http://localhost:8080/identity/connect/oidc-signin` (ou o path que o Vaultwarden espera — confirmar em `src/sso.rs`/`src/sso_client.rs`)
- Scopes: `openid`, `profile`, `email`, `offline_access`
- Claim `email` mapeado com domínio corporativo
- Confirmar MFA policy do tenant (Azure MFA já ativo — reusar)
- Documentar valores em `docs/prds/INF-04-azure-ad-oidc-app.md` (este arquivo) sem incluir secret

**Fora:**
- App Registration no tenant de produção
- Group claims / role mapping avançado (fica para POC-07/POC-08)
- Federação com outros IdPs

## Critérios de aceitação
- [x] App Registration criado no tenant Nova Corrente (`NewChainSafe PoC`)
- [x] Client ID registrado no `.env` local (não versionado)
- [x] Client Secret gerado (validade 90 dias), salvo no `.env` local
- [x] Redirect URI cadastrada: `http://localhost:8080/identity/connect/oidc-signin`
- [x] Scopes `openid profile email offline_access` configurados + admin consent
- [x] Tenant Authority URL registrado no `.env`
- [x] ID tokens habilitado em Authentication → Configurações
- [ ] Restrição de domínio `novacorrente.ind.br` planejada (implementação vai em POC-02)

## Dependências
- Acesso administrativo ao tenant Azure AD de teste da Nova Corrente

## Riscos & mitigações
- **Vazamento do Client Secret** → prazo curto, rotação planejada, nunca em `git`.
- **Redirect URI errado** → validar no código-fonte (`src/sso.rs`) antes de cadastrar.
- **MFA policy divergente entre tenants** → confirmar que tenant de teste replica policy de produção antes de validar POC-03.

## Runbook (executar no portal Azure)

**Confirmado no código-fonte** (`src/config.rs:1359` → `generate_sso_callback_path`):
- Redirect URI PoC local: `http://localhost:8080/identity/connect/oidc-signin`
- Formato geral: `<DOMAIN>/identity/connect/oidc-signin`

Passos:

1. Portal Azure → Microsoft Entra ID → **App registrations** → **New registration**
2. **Name:** `NewChainSafe PoC`
3. **Supported account types:** `Accounts in this organizational directory only` (single tenant)
4. **Redirect URI:**
   - Platform: **Web**
   - URI: `http://localhost:8080/identity/connect/oidc-signin`
5. Registrar → anotar:
   - **Application (client) ID** → vai em `SSO_CLIENT_ID`
   - **Directory (tenant) ID** → compõe `SSO_AUTHORITY=https://login.microsoftonline.com/<tenant-id>/v2.0`
6. **Certificates & secrets** → **New client secret**
   - Description: `NewChainSafe PoC secret`
   - Expires: **90 days** (rotação obrigatória)
   - Copiar **Value** (não Secret ID) → vai em `SSO_CLIENT_SECRET` no `.env` local
   - **Nunca commitar**. Nunca sair do portal antes de copiar (valor não é exibido novamente).
7. **API permissions** → **Microsoft Graph** → **Delegated**:
   - `openid`
   - `profile`
   - `email`
   - `offline_access`
   - Grant admin consent (se policy do tenant exigir)
8. **Token configuration** (opcional, ajuda debug):
   - Add optional claim → ID token → `email`, `preferred_username`
9. **Authentication** → confirmar que **ID tokens** está marcado em "Implicit grant and hybrid flows" (Vaultwarden usa authorization code + PKCE, mas Azure exige o toggle para emitir ID token no callback).

Valores a preencher no `.env` local após registro:
```
SSO_ENABLED=true
SSO_ONLY=false
SSO_CLIENT_ID=<application-client-id>
SSO_CLIENT_SECRET=<secret-value>
SSO_AUTHORITY=https://login.microsoftonline.com/<tenant-id>/v2.0
SSO_SCOPES=email profile
SIGNUPS_DOMAINS_WHITELIST=novacorrente.ind.br
```

## Referências
- `src/sso.rs`, `src/sso_client.rs`, `src/api/identity.rs:1216` (rota `/identity/connect/oidc-signin`)
- `src/config.rs:1359` (`generate_sso_callback_path`)
- Docs Vaultwarden SSO: `.env.template` — seção SSO
- Azure AD App Registrations: https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app
