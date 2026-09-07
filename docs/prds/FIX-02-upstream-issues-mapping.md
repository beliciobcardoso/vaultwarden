# FIX-02 — Mapear demais issues abertas relevantes no upstream

**Status:** todo · **Prioridade:** P2 · **Backlog:** [FIX-02](../backlog.md)

## Contexto
Além do BUG-01, o upstream pode ter outros bugs conhecidos relevantes para o cenário da Nova Corrente (SSO Azure AD, Postgres, Organizations, MFA). Mapear reduz surpresa em produção e alimenta priorização.

## Problema
Sem inventário, tomamos decisões cegas. Uma issue crítica não conhecida pode virar bloqueio tarde no ciclo.

## Objetivo
Uma lista curada em `docs/upstream-issues.md` (a criar) com issues abertas relevantes, cada uma classificada por severidade e impacto no PoC/produção.

## Escopo
**Dentro:**
- Buscar issues em `github.com/dani-garcia/vaultwarden/issues` filtrando: `sso`, `azure`, `postgres`, `oidc`, `organization`, `mfa`, `webauthn`
- Ler triagem existente (labels `bug`, `waiting-triage`)
- Selecionar as top-10 mais relevantes ao contexto Nova Corrente
- Registrar cada uma com: título, link, severidade (nossa avaliação), impacto no PoC/produção, workaround conhecido
- Documento vive em `docs/upstream-issues.md`

**Fora:**
- Reproduzir cada issue (fica para PRDs dedicados se necessário)
- Propor fixes (idem)

## Critérios de aceitação
- [ ] `docs/upstream-issues.md` criado
- [ ] Mínimo 10 issues catalogadas (ou justificativa se houver menos relevantes)
- [ ] Cada entrada: título, link, severidade, impacto, workaround
- [ ] Referência cruzada: BUG-01 deve aparecer com link para issue upstream (mesma que FIX-01 identificar)

## Dependências
Nenhuma dura; complementa FIX-01.

## Riscos & mitigações
- **Volume alto de issues** → time-box de 2h de leitura; qualidade > quantidade.
- **Documento vira desatualizado** → nota indicando data da última revisão; revisar antes de decisões grandes.

## Referências
- FIX-01
- Upstream issues: https://github.com/dani-garcia/vaultwarden/issues
