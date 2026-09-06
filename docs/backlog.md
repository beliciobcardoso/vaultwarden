# NewChainSafe — Backlog

**Projeto:** Cofre corporativo de senhas self-hosted (Nova Corrente)
**Plataforma base:** Vaultwarden (AGPL v3) — ver [market-analysis.md](./market-analysis.md) e [overview.md](./overview.md)
**Fork:** https://github.com/beliciobcardoso/vaultwarden
**Branch de trabalho:** `developer` (base para todo desenvolvimento)

---

## Restrição-mestra (hard constraint)

> **Nenhum escopo de deploy em produção** até a implementação estar concluída e validada localmente.
> Todo trabalho ocorre em PoC local (Docker Compose). Deploy em produção só entra em pauta após confirmação explícita.

---

## Legenda

- Status: `todo` · `doing` · `blocked` · `done`
- Prioridade: `P0` (crítico) · `P1` (alto) · `P2` (médio) · `P3` (baixo)

---

## 1. Setup & Infraestrutura Local

| ID | Tarefa | Prioridade | Status |
|----|--------|-----------|--------|
| INF-01 | Criar branch `developer` no fork | P0 | done |
| INF-02 | Docker Compose: Vaultwarden + PostgreSQL | P0 | todo |
| INF-03 | Configurar variáveis de ambiente (`.env`) — sem segredos versionados | P0 | todo |
| INF-04 | Registrar aplicação OIDC no Azure AD (tenant de teste) | P0 | todo |
| INF-05 | Configurar SSO Azure AD (OIDC) modo padrão — **NÃO** `SSO_ONLY` | P0 | todo |

---

## 2. Bugs Conhecidos

| ID | Bug | Prioridade | Status | Detalhe / Workaround |
|----|-----|-----------|--------|----------------------|
| BUG-01 | `SSO_ONLY` + Azure AD → redirect loop | P0 | blocked | Não ativar `SSO_ONLY` até resolução upstream. Investigar causa raiz e propor fix no fork. |

> Bugs adicionais encontrados durante a PoC devem ser registrados aqui.

---

## 3. PoC — Validação Funcional

| ID | Tarefa | Prioridade | Status |
|----|--------|-----------|--------|
| POC-01 | Subir stack local e autenticar via Azure AD (SSO padrão) | P0 | todo |
| POC-02 | Validar restrição de domínio `novacorrente.ind.br` | P0 | todo |
| POC-03 | Validar MFA step-up (Azure MFA na visualização de segredo, não só login) | P0 | todo |
| POC-04 | Validar criptografia E2E (servidor/DB nunca veem texto plano) | P0 | todo |
| POC-05 | Testar extensão de navegador Bitwarden conectada ao Vaultwarden local | P1 | todo |
| POC-06 | Testar cliente desktop Bitwarden conectado ao Vaultwarden local | P1 | todo |
| POC-07 | Definir e testar política de grupos e compartilhamento | P1 | todo |
| POC-08 | Validar controle granular de permissão por grupo (leitura/escrita) | P2 | todo |

---

## 4. Correção de Falhas no Fork

| ID | Tarefa | Prioridade | Status |
|----|--------|-----------|--------|
| FIX-01 | Investigar causa raiz do BUG-01 (`SSO_ONLY` redirect loop) | P1 | todo |
| FIX-02 | Mapear demais issues abertas relevantes no upstream | P2 | todo |

---

## 5. Riscos & Mitigações

| Risco | Mitigação |
|-------|-----------|
| Bug `SSO_ONLY` + Azure AD | Usar SSO padrão; não ativar `SSO_ONLY` até fix |
| Bus factor alto (projeto ~1 committer principal) | Monitorar atividade; migração p/ Bitwarden Enterprise possível sem perda de dados |

---

## 6. Fora de Escopo (por ora)

- Deploy em produção (bloqueado por restrição-mestra)
- Hardening de infra produtiva (TLS termination, backup, HA)
- Integração com pipeline CI/CD

---

**Status geral:** PoC em planejamento. Setup local pendente.
