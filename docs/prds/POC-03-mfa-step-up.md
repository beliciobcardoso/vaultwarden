# POC-03 — Validar MFA step-up (Azure MFA na visualização de segredo)

**Status:** todo · **Prioridade:** P0 · **Backlog:** [POC-03](../backlog.md)

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
- [ ] Documentado o comportamento real de step-up com Vaultwarden + Bitwarden client + Azure AD
- [ ] Testado que Azure MFA dispara no login SSO
- [ ] Master password validada como camada de proteção do vault
- [ ] Timeout curto (`VAULT_TIMEOUT_MINUTES`) configurado e testado
- [ ] Gap entre requisito ("MFA na visualização de cada segredo") e realidade explicitamente registrado — com decisão: aceitar, endurecer via timeout, ou entrar novo item no backlog para custom step-up

## Dependências
POC-01, INF-05

## Riscos & mitigações
- **Requisito não atendido nativamente** → registrar gap, discutir com stakeholder; possíveis mitigações: timeout curto, log de auditoria (eventos), master password forte obrigatória.
- **Conditional Access mal configurada** → validar policy no tenant de teste.

## Referências
- `src/sso.rs` (claims processing)
- Docs Bitwarden: Vault Timeout, Master Password
