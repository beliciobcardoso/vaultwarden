# FIX-01 — Investigar causa raiz do BUG-01 (`SSO_ONLY` redirect loop)

**Status:** todo · **Prioridade:** P1 · **Backlog:** [FIX-01](../backlog.md)

## Contexto
BUG-01 mantém `SSO_ONLY=false` como workaround, mas o modo desejável em produção é `SSO_ONLY=true`. Precisa-se de investigação técnica que produza: (a) diagnóstico da causa raiz, (b) issue upstream (ou identificação da existente), (c) proposta de patch — testada no fork antes de tentar upstream.

## Problema
Ativar `SSO_ONLY=true` com Azure AD como IdP dispara loop de redirect entre Vaultwarden e Microsoft. Sintoma sugere: (a) cookie de sessão não sendo setado, (b) parâmetro OIDC divergente esperado por Azure, ou (c) lógica de guard em `SSO_ONLY` desviando cedo demais no callback.

## Objetivo
Diagnóstico documentado + PR no fork (branch dedicada a partir de `developer`) com fix ou workaround estruturado. Se aplicável, PR upstream a partir de branch isolada de `main`.

## Escopo
**Dentro:**
- Reproduzir bug em ambiente local com `SSO_ONLY=true` (usar stack do INF-02 e config do INF-05, com override do `.env`)
- Coletar logs Vaultwarden (`RUST_LOG=vaultwarden=trace,vaultwarden::sso=trace`) e trace do navegador (network tab, cookies)
- Ler `src/sso.rs`, `src/sso_client.rs`, `src/api/identity.rs` — mapear fluxo de callback
- Identificar issue upstream correspondente (buscar em `github.com/dani-garcia/vaultwarden/issues?q=SSO_ONLY+azure`)
- Se issue não existe, abrir uma com repro mínimo
- Escrever patch candidato no fork (branch `fix/FIX-01-sso-only-loop`)
- Rodar testes (`cargo test --features postgresql`) + Playwright SSO spec
- Documentar tudo neste PRD

**Fora:**
- Merge no upstream (depende de review do maintainer; nosso ciclo termina no PR)
- Refatoração ampla de `src/sso.rs`

## Critérios de aceitação
- [ ] Bug reproduzido localmente com evidências (log + trace de rede)
- [ ] Causa raiz identificada e explicada em uma seção deste PRD
- [ ] Issue upstream referenciada ou aberta
- [ ] Patch candidato existe em branch `fix/FIX-01-sso-only-loop` sobre `developer`
- [ ] Testes locais passam (`cargo test`, Playwright)
- [ ] Com `SSO_ONLY=true`, login com Azure AD completa sem loop
- [ ] PRD atualizado com decisão sobre PR upstream (abrir agora / esperar mais casos / abandonar)

## Dependências
- INF-02, INF-05 (para reprodução)
- BUG-01 (referência)

## Riscos & mitigações
- **Bug depender de config específica do tenant Azure AD** → testar em App Registration nova, comparar; documentar.
- **Fix requerer alteração invasiva** → discutir com stakeholder antes de investir; alternativa é manter `SSO_ONLY=false` + `SIGNUPS_ALLOWED=false` como workaround permanente.
- **Regressão em outros IdPs** → rodar Playwright completo, não só suite SSO.

## Referências
- BUG-01
- `src/sso.rs`, `src/sso_client.rs`, `src/api/identity.rs`
- Upstream issues: a linkar após busca
