# POC-07 — Definir e testar política de grupos e compartilhamento

**Status:** done · **Prioridade:** P1 · **Backlog:** [POC-07](../backlog.md)

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
- [x] Modelo Org/Collection/Group documentado (ver "Política adotada" abaixo)
- [x] Org "Nova Corrente" criada no PoC (owner: `suporte@novacorrente.ind.br`)
- [x] 2 Collections criadas: `TI-Infra` e `Financeiro`
- [x] 2 usuários de teste funcionais via SSO (`belicio.cardoso@` e `suporte@novacorrente.ind.br`)
- [x] Item compartilhado visível a quem tem acesso, invisível a quem não tem — validado empiricamente
- [x] Política escrita em `docs/` referenciada aqui (seção "Política adotada")

## Dependências
POC-01, POC-02, INF-08 (SMTP para envio de convite)

## Riscos & mitigações
- **Confusão terminológica com stakeholder** → glossário no docs (usuário fala "grupo", nós mapeamos para Collection).
- **Permissão de admin de Org confusa** → seguir docs Bitwarden; documentar quem é Owner/Admin/Manager.

## Política adotada — Nova Corrente

**Estrutura escolhida:** uma única Organization `Nova Corrente` contendo várias Collections por área/projeto. Justificativa:
- Facilita administração (um único painel de membros e políticas).
- Colaboração cruzada entre áreas é comum na Nova Corrente — múltiplas Orgs criariam atrito.
- Permite políticas server-side (2FA obrigatório, master password mínima) uniformes.
- Modelo é reversível: se uma área precisar de isolamento total, dá para criar Org separada no futuro sem migrar as demais.

**Mapeamento terminológico (glossário para stakeholder):**

| Negócio (Nova Corrente) | Bitwarden/Vaultwarden |
|-------------------------|------------------------|
| Grupo de segredos       | Collection             |
| Papel/departamento      | Group (opcional, agrupa membros para atribuição em lote de Collections) |
| Empresa                 | Organization           |
| Segredo individual      | Item (Cipher)          |

**Papéis (Roles):**
- **Proprietário (Owner):** conta administrativa do vault. Nova Corrente: `suporte@novacorrente.ind.br` (ver ressalva abaixo).
- **Administrador (Admin):** gestão de membros/collections, sem poder de deletar a Org.
- **Gerente (Manager):** administra Collections específicas às quais tem acesso.
- **Usuário (User):** consumidor final; só vê Collections atribuídas.

**Ressalva sobre owner:** conta owner deve ser **compartilhada/institucional** (não pessoal). `suporte@novacorrente.ind.br` atende — não vai embora se um colaborador sair. Regra: nunca ter uma única pessoa como owner solo.

**Convenção de nomes de Collection:**
- Formato: `<Área>-<SubÁrea opcional>` (kebab evitado para melhor legibilidade em UI). Ex.: `TI-Infra`, `TI-Aplicações`, `Financeiro`, `Financeiro-Contas`, `RH-Sistemas`.
- Criar Collection nova precisa aprovação do owner/admin (evita fragmentação).

**Sync de grupos Azure AD:** fora de escopo. Vaultwarden não implementa Directory Connector (Bitwarden Enterprise-only). Manter atribuição manual até PoC amadurecer. Reavaliar em ITER-2.

## Evidências (2026-09-08)

1. **Org criada:** `Nova Corrente` (owner: `suporte@novacorrente.ind.br`).
2. **Collections:** `TI-Infra`, `Financeiro`, mais `Conjunto padrão` (default do Vaultwarden — não usar em produção; renomear ou remover em rollout).
3. **Membros:**
   - `suporte@novacorrente.ind.br` → role: Proprietário → acesso: **apenas** `Financeiro`
   - `belicio.cardoso@novacorrente.ind.br` → role: Usuário → acesso: **apenas** `TI-Infra`
4. **Convite:** enviado via SMTP MailDev (INF-08). Emails no inbox `http://localhost:1080`:
   - `Join Nova Corrente` → belicio (aceito)
   - `Invitation to Nova Corrente accepted` → suporte (confirmação)
5. **Itens de teste:**
   - `acesso-banco-teste` em `Financeiro` (criado por suporte)
   - `senha-servidor-teste` em `TI-Infra` (criado por belicio)
6. **Isolamento validado:**
   - belicio vê `senha-servidor-teste`, NÃO vê `acesso-banco-teste`
   - suporte vê `acesso-banco-teste`, NÃO vê `senha-servidor-teste`

## Ações de acompanhamento

- Em rollout: renomear/remover `Conjunto padrão` (nome genérico atrapalha convenção).
- POC-08: validar controle granular de permissão (read-only, edit, manage) dentro da mesma Collection.
- Habilitar policy `TwoFactorAuthentication` na Org (decisão de POC-03) quando membros reais forem convidados.
- Definir processo formal de solicitação de nova Collection (ticket + aprovação de owner) — fora deste PRD, entra em política operacional.

## Referências
- `src/api/core/organizations.rs`
- `src/db/models/org_policy.rs`
- POC-03 (policy `TwoFactorAuthentication` a habilitar aqui)
- INF-08 (SMTP para convites)
- Docs Bitwarden: Organizations, Collections, Groups
