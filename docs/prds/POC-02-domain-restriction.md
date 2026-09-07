# POC-02 — Validar restrição de domínio `novacorrente.ind.br`

**Status:** todo · **Prioridade:** P0 · **Backlog:** [POC-02](../backlog.md)

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
- [ ] `SIGNUPS_DOMAINS_WHITELIST=novacorrente.ind.br` no `.env`
- [ ] Usuário `@novacorrente.ind.br` completa login SSO
- [ ] Usuário fora do domínio (ex.: `@gmail.com` via guest) é rejeitado
- [ ] Mensagem de erro é razoável (não vaza detalhes internos)
- [ ] Comportamento consistente entre nova sessão anônima e sessão existente

## Dependências
POC-01

## Riscos & mitigações
- **Whitelist não aplicada em SSO** → validar código em `src/sso.rs` / `src/api/identity.rs`; se não cobrir SSO, abrir bug e propor patch.
- **Guest Azure AD passar** → confirmar policy do tenant; se necessário, endurecer no Azure AD.

## Referências
- `src/sso.rs`
- `.env.template` (chave `SIGNUPS_DOMAINS_WHITELIST`)
