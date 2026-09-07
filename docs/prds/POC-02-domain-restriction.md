# POC-02 — Validar restrição de domínio `novacorrente.ind.br`

**Status:** done · **Prioridade:** P0 · **Backlog:** [POC-02](../backlog.md)

## Contexto
Cofre corporativo deve aceitar **apenas** contas `@novacorrente.ind.br`. Um usuário externo autenticando pelo Azure AD (guest, outro tenant, conta pessoal) precisa ser barrado.

## Problema
Vaultwarden não faz restrição por domínio de email por default. Precisa ser implementada por: (a) configuração do App Registration Azure AD, ou (b) whitelist no Vaultwarden (`ORG_GROUPS_ENABLED`/`SIGNUPS_DOMAINS_WHITELIST`), ou (c) ambos.

## Objetivo
Provar que um usuário fora do domínio corporativo é rejeitado no fluxo SSO, e um usuário do domínio é aceito.

## Escopo
**Dentro:**
- Configurar `SIGNUPS_DOMAINS_WHITELIST=novacorrente.ind.br` no `.env`
- Confirmar comportamento do Azure AD (App Registration configurada para "Accounts in this organizational directory only" idealmente)
- Testar 2 cenários: usuário do domínio (aceito) + usuário fora do domínio (rejeitado)
- Documentar mensagem de erro apresentada ao usuário rejeitado

**Fora:**
- Restrição por grupo Azure AD (POC-07)
- Login local (bloqueado por `SIGNUPS_ALLOWED=false`)

## Critérios de aceitação
- [x] `SIGNUPS_DOMAINS_WHITELIST=novacorrente.ind.br` no `.env` (carregado no container, `docker compose exec vaultwarden env | grep SIGNUPS_DOMAINS_WHITELIST` confirma)
- [x] Usuário `@novacorrente.ind.br` completa login SSO (validado em POC-01 com `belicio.cardoso@novacorrente.ind.br`)
- [x] Usuário fora do domínio é rejeitado — **validado por leitura de código** (ver seção "Validação por code review")
- [x] Mensagem de erro é razoável (`"Email domain not allowed"` — sem vazamento de detalhes internos)
- [x] Comportamento consistente entre nova sessão anônima e sessão existente (fluxo passa por `is_sso_signup_allowed` no callback, independente da sessão prévia)

## Dependências
POC-01

## Riscos & mitigações
- **Whitelist não aplicada em SSO** → validar código em `src/sso.rs` / `src/api/identity.rs`; se não cobrir SSO, abrir bug e propor patch.
- **Guest Azure AD passar** → confirmar policy do tenant; se necessário, endurecer no Azure AD.

## Validação por code review

Cenário negativo (usuário fora do domínio) não testado end-to-end por indisponibilidade de conta guest/tenant externo. Validado por leitura do código:

1. **`src/config.rs:1526-1536`** — `is_email_domain_allowed(email)`:
   - Parseia domínio pós-`@`.
   - Retorna `true` se whitelist vazia OU domínio bate com algum item da lista.
   - Comparação case-insensitive (email é `.to_lowercase()`), whitelist tem `.trim()`.
   - Email malformado → warn + `false` (fail-closed).

2. **`src/config.rs:1551-1558`** — `is_sso_signup_allowed(email)`:
   - Se whitelist vazia → cai em `sso_signups_allowed()` (bool global).
   - Se whitelist preenchida → **override**, delega a `is_email_domain_allowed`.
   - Nossa config: `SIGNUPS_DOMAINS_WHITELIST=novacorrente.ind.br` → override ativo.

3. **`src/api/identity.rs:302-316`** — callback SSO, ramo `None` (usuário novo):
   ```rust
   if !CONFIG.is_sso_signup_allowed(&user_infos.email) {
       if CONFIG.signups_domains_whitelist().is_empty() {
           err!("Signups are disabled. You will need an invitation", ...);
       }
       err!("Email domain not allowed", ...);
   }
   ```
   Whitelist preenchida → mensagem retornada é `"Email domain not allowed"`, sem vazar valor da whitelist nem detalhes de config.

**Conclusão:** com `SIGNUPS_DOMAINS_WHITELIST=novacorrente.ind.br`, qualquer email cujo domínio (após `@`, case-insensitive, trimmed) ≠ `novacorrente.ind.br` é bloqueado no callback SSO com mensagem genérica. Evento `UserFailedLogIn` é emitido (observável em logs/`/admin`).

**Limitação conhecida:** usuário Azure AD guest com email `@novacorrente.ind.br` **passa**, pois a checagem é por string do email, não por origem no tenant. Endurecer no Azure AD (Conditional Access ou restringir "supported account types" da App Registration) fica fora deste PRD — nossa App Registration já é "single tenant" (INF-04 passo 3), o que reduz o risco.

## Referências
- `src/api/identity.rs:302-316` (callback SSO — enforce)
- `src/config.rs:1526-1558` (helpers de whitelist)
- `.env.template` (chave `SIGNUPS_DOMAINS_WHITELIST`)
- POC-01 (validação positiva)
