# BUG-01 — `SSO_ONLY` + Azure AD → redirect loop

**Status:** blocked · **Prioridade:** P0 · **Backlog:** [BUG-01](../backlog.md)

## Contexto
Bug conhecido do Vaultwarden: ao ativar `SSO_ONLY=true` com Azure AD como IdP, o navegador entra em loop entre Vaultwarden e o login da Microsoft, sem completar a sessão. Reportado no repositório upstream.

## Problema
`SSO_ONLY=true` é o modo desejável em produção (elimina login local, força fluxo corporativo), mas hoje quebra com Azure AD. Enquanto não houver fix, PoC/produção precisam ficar em SSO padrão — o que deixa a porta do login local aberta e obriga a compensar com `SIGNUPS_ALLOWED=false`.

## Objetivo
Este PRD é um **placeholder de rastreio** do bug. A investigação e a proposta de fix vão em **FIX-01**. O status permanece `blocked` até que FIX-01 conclua ou o upstream publique fix.

## Escopo
**Dentro:**
- Manter registro do bug, sintomas e workaround
- Referenciar FIX-01 para a investigação técnica
- Manter regra `SSO_ONLY=false` explícita em INF-05

**Fora:**
- Correção efetiva (vai em FIX-01)

## Critérios de aceitação (deste PRD)
- [x] Bug documentado em `docs/backlog.md` e `docs/overview.md`
- [x] Workaround (`SSO_ONLY=false`) aplicado em INF-05
- [ ] Issue upstream identificada e linkada aqui (a preencher em FIX-01)
- [ ] Fix aplicado (upstream ou fork) → então este PRD passa para `done`

## Dependências
- FIX-01 (investigação e proposta de fix)

## Riscos & mitigações
- **Ativação acidental de `SSO_ONLY`** → variável explícita no `.env` + comentário citando este PRD.
- **Fix upstream demorar** → migração para Bitwarden Enterprise permanece viável (bus factor citado em `overview.md`).

## Sintomas conhecidos
- Após clique em "Login with SSO", redirect vai para Azure AD, autentica, retorna a Vaultwarden, e recomeça o ciclo sem estabelecer sessão.

## Referências
- FIX-01
- INF-05
- `src/sso.rs`, `src/sso_client.rs`, `src/api/identity.rs`
- Upstream: `github.com/dani-garcia/vaultwarden` (issue a linkar em FIX-01)
