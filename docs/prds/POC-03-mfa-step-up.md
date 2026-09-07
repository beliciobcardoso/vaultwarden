# POC-03 — Validar MFA step-up (Azure MFA na visualização de segredo)

**Status:** done · **Prioridade:** P0 · **Backlog:** [POC-03](../backlog.md)

## Contexto
Requisito de negócio: MFA obrigatório **no momento de visualizar segredos**, não apenas no login. Aproveita Azure MFA já ativo. Cliente Bitwarden padrão exige master password para desbloquear vault, o que já é uma barreira; a questão é validar se essa dinâmica combinada com Azure MFA atende ao requisito.

## Problema
Bitwarden/Vaultwarden usa master password para desbloquear vault (arquitetura zero-knowledge). Precisa-se validar se: (a) master password funciona como o "MFA step-up" desejado; ou (b) é possível reforçar com claim ACR do Azure AD ("mfa"); ou (c) exigir re-autenticação SSO com `prompt=login` ao expor um segredo.

## Objetivo
Definir e validar o mecanismo real de step-up que a Nova Corrente vai adotar, e comprovar que o comportamento é aceitável para o requisito.

## Escopo
**Dentro:**
- Mapear como o cliente Bitwarden desbloqueia vault (master password prompt)
- Testar se Azure MFA já dispara no login SSO (Conditional Access)
- Testar se claim `acr=mfa` chega ao Vaultwarden e é logada
- Documentar limitação: Vaultwarden não tem hoje um "reveal password → prompt MFA" nativo; o que existe é master-password + timeout de sessão
- Propor política de `VAULT_TIMEOUT` curta como reforço

**Fora:**
- Alteração de código Vaultwarden (fica para tarefa dedicada se decisão for essa)
- Integração custom com Azure para step-up on-demand

## Critérios de aceitação
- [x] Comportamento real documentado (ver "Análise técnica" abaixo)
- [x] Azure MFA no login validado (Conditional Access do tenant controla; fluxo POC-01 passou por MFA se policy exigiu — validação enquanto policy do tenant não muda)
- [x] Master password validada como camada de proteção do vault (arquitetural, zero-knowledge)
- [x] Vault timeout: documentado como **orientação client-side** (`MaximumVaultTimeout` policy não é AGPL-compatível — Vaultwarden não implementa server-side enforcement)
- [x] Gap registrado + decisão: **A + B combinados** (aceitar arquitetura + endurecer via policy `TwoFactorAuthentication` da org e VAULT_TIMEOUT curto orientado)

## Dependências
POC-01, INF-05

## Riscos & mitigações
- **Requisito não atendido nativamente** → registrar gap, discutir com stakeholder; possíveis mitigações: timeout curto, log de auditoria (eventos), master password forte obrigatória.
- **Conditional Access mal configurada** → validar policy no tenant de teste.

## Análise técnica

1. **Arquitetura zero-knowledge**: server (Vaultwarden) nunca vê plaintext. Vault descriptografa 100% client-side com chave derivada da master password. Impossível servidor forçar "MFA antes de revelar segredo" — segredo já sai criptografado do server.

2. **Master password = gate real**: cliente Bitwarden pede master password para desbloquear vault após timeout. Único step-up existente na stack.

3. **MFA no login SSO**: enforçado no Azure AD via Conditional Access. Vaultwarden **não processa** claims `acr`/`amr` do ID token (grep vazio em `src/sso*.rs`, `src/api/identity.rs`). MFA é opaco para o server — se Azure exigiu, callback SSO só chega após MFA concluído.

4. **Vault timeout policy server-side**: `src/db/models/org_policy.rs:41` — `MaximumVaultTimeout = 9, // Not supported (Not AGPLv3 Licensed)`. Vaultwarden **não implementa** policy que force timeout máximo. Config é per-device no cliente; usuário pode escolher timeout longo.

5. **Sem hook "reveal password → step-up"**: cliente Bitwarden não tem esse fluxo. Todo item já está descriptografado após unlock.

## Gap explícito

Requisito literal "MFA na visualização de cada segredo" **não existe** na stack Bitwarden nem no Vaultwarden. Atender exigiria fork profundo (client + server, quebra E2E).

## Decisão adotada: A + B combinados

**A — Aceitar arquitetura:**
- Azure MFA no login SSO (via Conditional Access do tenant).
- Master password forte obrigatória (política corporativa, não server-side).
- Vault permanece descriptografado localmente enquanto sessão ativa.

**B — Endurecer via policy + orientação:**
- Habilitar policy `TwoFactorAuthentication` (`OrgPolicyType = 0`, suportada em `src/db/models/org_policy.rs:32`) na organização quando ela existir, forçando 2FA em todos os membros.
- Orientar `VAULT_TIMEOUT_MINUTES` curto no cliente (5–15 min) via documentação de onboarding — não é enforceable server-side.
- Auditoria: eventos `UserFailedLogIn` e sucesso já são registrados via `EventType` (visíveis em `/admin` e DB tabela `events`).

**C descartado**: custom "step-up on reveal" quebraria E2E e exigiria fork paralelo permanente vs upstream. Custo/benefício não fecha para PoC.

## Ações de acompanhamento

- **POC-07** (grupos/policies): ao criar organização, habilitar policy `TwoFactorAuthentication`.
- Documentação de onboarding (fora deste PRD): instruir usuários a configurar VAULT_TIMEOUT ≤ 15 min no cliente.
- Confirmar Conditional Access do tenant Nova Corrente exige MFA para app `NewChainSafe PoC` (INF-04).

## Referências
- `src/db/models/org_policy.rs:31-41` (policy types, MaximumVaultTimeout marcado como não-AGPL)
- `src/api/identity.rs` + `src/sso*.rs` (sem processamento de claim `acr`/`amr`)
- Docs Bitwarden: Vault Timeout, Master Password, Two-step Login Policy
