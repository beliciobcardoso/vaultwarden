# POC-07 — Definir e testar política de grupos e compartilhamento

**Status:** todo · **Prioridade:** P1 · **Backlog:** [POC-07](../backlog.md)

## Contexto
Requisito: usuários organizam segredos em **grupos** e podem compartilhar grupos com colegas do mesmo domínio. No modelo Bitwarden, isso se mapeia para **Organizations → Collections → Groups**.

## Problema
Nomenclatura de negócio ("grupos") não bate 1:1 com Bitwarden. Precisa-se decidir:
- Uma única Organization Nova Corrente contendo todas as Collections?
- Vs. Organizations por departamento?
- Como Groups (Bitwarden) se relacionam com Groups Azure AD (opcional via `SSO`)?

## Objetivo
Modelo de organização definido, criado no PoC, testado com pelo menos 2 usuários compartilhando 1 grupo (Collection).

## Escopo
**Dentro:**
- Decidir estrutura: 1 Org "Nova Corrente" + Collections por área/projeto
- Criar Org, 2 Collections de exemplo (`TI-Infra`, `Financeiro`)
- Criar 2 usuários de teste via SSO
- Adicionar ambos à Org, cada um em Collections diferentes
- Compartilhar 1 item; validar visibilidade e proibição cruzada
- Documentar política em `docs/` (novo arquivo ou seção deste PRD)

**Fora:**
- Sync automático de grupos Azure AD (fica para tarefa futura se necessário)
- Directory Connector (recurso Bitwarden Enterprise; não aplicável)

## Critérios de aceitação
- [ ] Modelo Org/Collection/Group documentado
- [ ] Org "Nova Corrente" criada no PoC
- [ ] 2 Collections criadas
- [ ] 2 usuários de teste funcionais via SSO
- [ ] Item compartilhado visível a quem tem acesso, invisível a quem não tem
- [ ] Política escrita em `docs/` referenciada aqui

## Dependências
POC-01, POC-02

## Riscos & mitigações
- **Confusão terminológica com stakeholder** → glossário no docs (usuário fala "grupo", nós mapeamos para Collection).
- **Permissão de admin de Org confusa** → seguir docs Bitwarden; documentar quem é Owner/Admin/Manager.

## Referências
- `src/api/core/organizations.rs`
- Docs Bitwarden: Organizations, Collections, Groups
