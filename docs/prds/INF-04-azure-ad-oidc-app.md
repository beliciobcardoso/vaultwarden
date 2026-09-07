# INF-04 — Registrar aplicação OIDC no Azure AD (tenant de teste)

**Status:** todo · **Prioridade:** P0 · **Backlog:** [INF-04](../backlog.md)

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
- [ ] App Registration criado no tenant de teste
- [ ] Client ID registrado no `.env` local (não versionado)
- [ ] Client Secret gerado, prazo de validade documentado, salvo no `.env` local
- [ ] Redirect URI cadastrada e validada contra o path real do Vaultwarden
- [ ] Scopes `openid profile email offline_access` configurados
- [ ] Tenant Authority URL (`https://login.microsoftonline.com/<tenant-id>/v2.0`) registrado no `.env`
- [ ] Restrição de domínio `novacorrente.ind.br` planejada (implementação vai em POC-02)

## Dependências
- Acesso administrativo ao tenant Azure AD de teste da Nova Corrente

## Riscos & mitigações
- **Vazamento do Client Secret** → prazo curto, rotação planejada, nunca em `git`.
- **Redirect URI errado** → validar no código-fonte (`src/sso.rs`) antes de cadastrar.
- **MFA policy divergente entre tenants** → confirmar que tenant de teste replica policy de produção antes de validar POC-03.

## Referências
- `src/sso.rs`, `src/sso_client.rs`
- Docs Vaultwarden SSO: `.env.template` — seção SSO
- Azure AD App Registrations
