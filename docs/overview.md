# NewChainSafe — Visão Geral do Projeto

**Empresa:** Nova Corrente  
**Domínio:** novacorrente.ind.br  
**Categoria:** Cofre corporativo de senhas — self-hosted

---

## Objetivo

Centralizar e proteger credenciais corporativas em um cofre de senhas hospedado na infraestrutura da própria empresa, com acesso exclusivo para colaboradores autenticados via Azure AD e sem custo de infraestrutura adicional.

---

## Autenticação e Acesso

- **SSO via Microsoft Azure AD** — login restrito ao domínio `novacorrente.ind.br`
- Nenhuma conta local: toda autenticação passa pelo Azure AD
- **MFA obrigatório para visualizar segredos** — utiliza o Azure MFA já ativo para todos os usuários corporativos
- Acesso negado a usuários fora do domínio corporativo

---

## Funcionalidades Principais

### Gerenciamento de Credenciais
- Cadastro, edição e exclusão de senhas e segredos
- Organização em **grupos** definidos pelo usuário
- **Compartilhamento de grupos** com colegas do mesmo domínio
- Controle granular de permissão por grupo (leitura / escrita)

### Segurança
- **Criptografia ponta a ponta (E2E):** segredos criptografados no cliente antes de serem transmitidos e armazenados; o servidor e o banco de dados nunca têm acesso ao texto plano
- MFA exigido no momento de visualização do segredo (não apenas no login)
- Isolamento de dados por usuário e por grupo

### Interface do Usuário
- Aplicação web responsiva
- **Extensão de navegador** — preenchimento automático de credenciais
- **Aplicativo desktop** — acesso seguro via aplicação nativa para Windows/Linux/macOS

---

## Restrições e Premissas

| Item | Decisão |
|------|---------|
| Infraestrutura | Self-hosted na infra existente da Nova Corrente |
| Custo adicional de infra | Zero — objetivo explícito |
| Provedor de identidade | Azure AD (já contratado) |
| MFA | Azure MFA (já ativo para todos os usuários) |
| Domínio de acesso | Apenas `novacorrente.ind.br` |
| Construção do zero | Preferência por OSS existente — avaliar antes de implementar |

---

## Decisão Técnica

- **Plataforma base: Vaultwarden** (AGPL v3) — selecionado após avaliação de 5 candidatos OSS; único viável com SSO Azure AD gratuito (ver [market-analysis.md](./market-analysis.md))
- Backend: Vaultwarden (Rust)
- Banco de dados: PostgreSQL
- Autenticação: SSO via Azure AD (OIDC) — modo SSO padrão (não `SSO_ONLY`)
- Criptografia: zero-knowledge — clientes oficiais Bitwarden criptografam no client-side

### Risco Conhecido

| Risco | Detalhe | Mitigação |
|-------|---------|-----------|
| Bug `SSO_ONLY` + Azure AD | Redirect loop ao usar modo `SSO_ONLY` — issue aberta no repositório Vaultwarden | Usar SSO padrão; **não ativar `SSO_ONLY`** até bug resolvido |
| Bus factor alto | Projeto mantido majoritariamente por 1 committer principal | Monitorar atividade; migração para Bitwarden Enterprise possível sem perda de dados |

---

## Próximos Passos

- [ ] PoC Vaultwarden: Docker Compose + Azure AD OIDC + PostgreSQL
- [ ] Validar fluxo de MFA step-up (Azure MFA na visualização de segredo)
- [ ] Testar extensão Bitwarden conectada ao Vaultwarden
- [ ] Testar cliente desktop Bitwarden conectado ao Vaultwarden
- [ ] Definir política de grupos e compartilhamento

---

## Status

`Plataforma selecionada — Vaultwarden (AGPL v3). PoC em planejamento.`
