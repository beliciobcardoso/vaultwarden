# NewChainSafe — Análise de Mercado e Avaliação de OSS

**Data:** 2026-09-06  
**Status:** Decisão tomada — Vaultwarden selecionado como plataforma base

---

## Contexto

Antes de construir do zero, avaliar soluções OSS self-hosted que cubram os requisitos críticos:

| Requisito | Peso |
|-----------|------|
| SSO via Azure AD (OIDC) | Obrigatório |
| MFA step-up para visualização de segredo | Obrigatório |
| Zero-knowledge / criptografia client-side | Obrigatório |
| PostgreSQL | Obrigatório |
| Self-hosted sem custo de licença adicional | Desejável |
| Auditoria de segurança independente | Desejável |
| Extensão de navegador | Desejável |
| Aplicativo desktop | Desejável |

---

## Candidatos Avaliados

> **Critério de corte aplicado:** SSO via Azure AD deve ser gratuito na edição self-hosted. Candidatos que bloqueiam SSO atrás de licença paga foram eliminados.

### 1. Bitwarden (oficial) — ELIMINADO

**Motivo:** SSO com Azure AD requer licença Enterprise (~$5/usuário/mês). Sem ela, autenticação cai para contas locais — incompatível com o requisito de SSO-only.

---

### 2. Vaultwarden

- **Licença:** AGPL v3 (reimplementação não-oficial do servidor Bitwarden em Rust)
- **Stack:** Rust
- **GitHub stars:** ~65.3k
- **Azure AD SSO:** OIDC disponível gratuitamente — porém `SSO_ONLY` tem **bug conhecido de redirect loop com Azure AD** (issue aberta no repositório)
- **Zero-knowledge:** Sim — usa os clientes oficiais Bitwarden; criptografia idêntica
- **PostgreSQL:** Sim
- **Docker / K8s:** Docker; sem Helm chart oficial (manifests comunitários existem)
- **Auditoria:** Não — não coberto pelas auditorias do Bitwarden
- **Extensão / Desktop:** Sim — todos os clientes oficiais Bitwarden funcionam sem modificação
- **Consumo:** ~20 MB RAM em idle

**Pontos positivos:** zero custo; estrelas altíssimas (~65k); consome mínimo de recursos; SSO gratuito.  
**Riscos:** projeto mantido majoritariamente por um committer principal (bus factor alto); bug de SSO_ONLY + Azure AD não resolvido; sem auditoria formal.

---

### 3. Passbolt — ELIMINADO

**Motivo:** SSO via Azure AD (OIDC/OAuth2) disponível apenas na Pro Edition (~€4/usuário/mês). Community Edition não suporta SSO.

---

### 4. Psono — ELIMINADO

**Motivo:** SSO via Azure AD (SAML) requer Enterprise Edition (~€2/usuário/mês). Community Edition não suporta SSO.

---

### 5. Padloc

- **Licença:** AGPL v3
- **Stack:** TypeScript / Node.js
- **GitHub stars:** ~2.1k
- **Azure AD SSO:** Não confirmado — sem OIDC/SAML documentado para self-hosted
- **Zero-knowledge:** Sim — SRP (Secure Remote Password)
- **PostgreSQL:** **Não — SQLite apenas**
- **Docker / K8s:** Docker; sem Helm chart
- **Auditoria:** Não
- **Extensão / Desktop:** Parcial

**Decisão: ELIMINADO** — não atende PostgreSQL (requisito obrigatório) e não tem SSO Azure AD confirmado.

---

## Matriz Comparativa

Apenas Vaultwarden sobreviveu ao critério de corte.

| Dimensão | Vaultwarden |
|----------|-------------|
| Azure AD SSO gratuito | ⚠️ (bug SSO_ONLY; usar SSO padrão) |
| Zero-knowledge | ✅ |
| PostgreSQL | ✅ |
| Auditoria independente | ❌ |
| Extensão de navegador | ✅ (clientes oficiais Bitwarden) |
| App desktop | ✅ (clientes oficiais Bitwarden) |
| Custo SSO | Gratuito |
| Comunidade | Muito alta (~65k stars) |
| Bus factor | **Alto** (1 committer principal) |
| Sustentabilidade | Média |

---

## Decisão Recomendada

**Partir com Vaultwarden** (Cenário B) para a fase de prova de conceito:

1. Custo zero mantém o requisito de "zero custo adicional de infraestrutura"
2. Todos os clientes Bitwarden funcionam sem modificação (extensão + desktop)
3. PostgreSQL nativo
4. SSO OIDC com Azure AD funcionando (evitar `SSO_ONLY` até bug resolvido)
5. Migração para Bitwarden Enterprise possível sem perda de dados (mesmo protocolo de criptografia)

**Risco principal a monitorar:** bug `SSO_ONLY` + Azure AD. Testar no ambiente de homologação antes de produção.

---

## Próximos Passos

- [ ] PoC Vaultwarden: Docker Compose + Azure AD OIDC + PostgreSQL
- [ ] Validar fluxo de MFA step-up (Azure MFA na visualização de segredo)
- [ ] Testar extensão Bitwarden conectada ao Vaultwarden
- [ ] Testar cliente desktop Bitwarden conectado ao Vaultwarden
- [ ] Documentar bug SSO_ONLY no registro de riscos
- [ ] Definir política de grupos e compartilhamento
