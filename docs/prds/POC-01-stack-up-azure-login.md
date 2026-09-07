# POC-01 — Subir stack local e autenticar via Azure AD (SSO padrão)

**Status:** todo · **Prioridade:** P0 · **Backlog:** [POC-01](../backlog.md)

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
- [ ] Stack sobe sem erros
- [ ] Web vault acessível em `http://localhost:8080`
- [ ] Fluxo SSO completa com usuário `@novacorrente.ind.br`
- [ ] Sessão persiste após refresh
- [ ] Log Vaultwarden sem `ERROR`/`WARN` no fluxo SSO
- [ ] Roteiro reproduzível documentado (comandos + prints ou descrição)

## Dependências
INF-02, INF-03, INF-04, INF-05

## Riscos & mitigações
- **Falha por config errada** → checar INF-05 primeiro (variáveis SSO), depois logs.
- **Redirect loop mesmo em SSO padrão** → não deveria (BUG-01 é só `SSO_ONLY`), mas se ocorrer, tratar como novo bug e registrar no backlog.

## Referências
- INF-02..INF-05
