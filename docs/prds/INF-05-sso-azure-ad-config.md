# INF-05 — Configurar SSO Azure AD (OIDC) modo padrão

**Status:** todo · **Prioridade:** P0 · **Backlog:** [INF-05](../backlog.md)

## Contexto
Com App Registration pronto (INF-04) e stack local rodando (INF-02/INF-03), configurar o Vaultwarden para autenticar via Azure AD **em modo SSO padrão** — **jamais** `SSO_ONLY` (bloqueado por BUG-01).

## Problema
Vaultwarden precisa de várias variáveis SSO_* corretas e coerentes com a App Registration; erro num campo derruba todo o fluxo. Ativar `SSO_ONLY` por engano quebra login (redirect loop) — cenário do BUG-01.

## Objetivo
Login via Azure AD funcional em modo SSO padrão (usuário pode escolher SSO ou senha local, mas SIGNUPS_ALLOWED=false garante que só quem já existe entra).

## Escopo
**Dentro:**
- `SSO_ENABLED=true`
- `SSO_ONLY=false` (explícito no `.env` como salvaguarda)
- `SSO_CLIENT_ID`, `SSO_CLIENT_SECRET`, `SSO_AUTHORITY` preenchidos com valores do INF-04
- `SSO_SCOPES="email profile"` (openid é implícito)
- `SSO_SIGNUPS_ALLOWED=true` para permitir provisionamento no primeiro login (usuário Azure AD válido → conta local criada)
- Testar login com um usuário `@novacorrente.ind.br` (o domínio-lock efetivo entra em POC-02)
- Documentar comando para tail dos logs de SSO (`RUST_LOG=vaultwarden::sso=debug`)

**Fora:**
- `SSO_ONLY=true` (proibido — BUG-01)
- Restrição de domínio hardcoded (fica em POC-02)
- Group/role claims (fica em POC-07/08)

## Critérios de aceitação
- [ ] `.env` local com todas as vars `SSO_*` preenchidas e comentadas
- [ ] `SSO_ONLY=false` presente e explícito
- [ ] Botão "Log in with SSO" aparece na tela de login do web vault
- [ ] Login completo via Azure AD funciona (redirect ida → Azure → callback → sessão)
- [ ] Log do Vaultwarden mostra fluxo SSO sem warnings/erros
- [ ] Teste com usuário fora do domínio: login concluído, mas fica pendente para POC-02 aplicar restrição

## Dependências
- INF-02, INF-03 (stack + env)
- INF-04 (App Registration existente)

## Riscos & mitigações
- **Ativar `SSO_ONLY` por engano** → variável explícita `SSO_ONLY=false` no `.env`, comentário destacado citando BUG-01.
- **Client Secret expirar** → prazo documentado em INF-04, alerta manual.
- **Callback URI divergente** → validar contra `src/sso.rs` antes de cadastrar.

## Referências
- INF-04 (App Registration)
- BUG-01 (`SSO_ONLY` redirect loop)
- `src/sso.rs`, `src/sso_client.rs`, `src/api/identity.rs`
