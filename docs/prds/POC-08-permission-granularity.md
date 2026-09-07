# POC-08 — Validar controle granular de permissão por grupo (leitura/escrita)

**Status:** todo · **Prioridade:** P2 · **Backlog:** [POC-08](../backlog.md)

## Contexto
Requisito: cada usuário deve ter permissão granular por Collection — apenas leitura, leitura+escrita, ou sem acesso. Complementa POC-07.

## Problema
Bitwarden expõe roles (Owner/Admin/Manager/User) por Org e permissões (Can view / Can edit / Can manage) por Collection ou Group. Precisa-se mapear roles reais da Nova Corrente para esse modelo.

## Objetivo
Matriz de permissões definida e validada em pelo menos 3 combinações (RO, RW, sem acesso) com usuários reais no PoC.

## Escopo
**Dentro:**
- Definir matriz de permissões esperada (por perfil de usuário Nova Corrente)
- Configurar 3 usuários com permissões distintas em Collections criadas em POC-07
- Testar cenários: leitura, tentativa de escrita negada, tentativa de acesso a Collection restrita negada
- Documentar matriz e resultados

**Fora:**
- Auditoria formal de RBAC
- Sincronização com Azure AD groups (fora do PoC)

## Critérios de aceitação
- [ ] Matriz de permissões documentada
- [ ] 3 usuários de teste com permissões distintas
- [ ] Cenário RO validado (edição bloqueada com mensagem clara)
- [ ] Cenário RW validado (edição persiste)
- [ ] Cenário sem acesso validado (Collection sequer aparece)
- [ ] Comportamento consistente entre web vault, extensão e desktop

## Dependências
POC-07

## Riscos & mitigações
- **Bug de permissão** (linhas visíveis onde não deveriam) → testar API direta (`curl`) além da UI para descartar cache do client.
- **Roles Bitwarden não cobrirem caso real** → registrar gap e propor workaround (múltiplas Collections).

## Referências
- `src/api/core/organizations.rs`
- `src/db/models/` (`organization.rs`, `collection.rs`, `user_organization.rs`)
