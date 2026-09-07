# POC-04 — Validar criptografia E2E (servidor/DB nunca veem texto plano)

**Status:** todo · **Prioridade:** P0 · **Backlog:** [POC-04](../backlog.md)

## Contexto
Bitwarden é zero-knowledge por design: cifragem/decifragem acontecem no cliente; servidor armazena apenas ciphertext + envelope de chaves derivadas da master password. Precisa-se comprovar isso na prática no PoC.

## Problema
Confiar no design não basta — precisa-se **provar** que o PostgreSQL não guarda plaintext e que o binário do servidor não expõe rota que decifre item de vault.

## Objetivo
Evidência empírica (query SQL + análise de código) de que segredos ficam cifrados em repouso e em trânsito para o servidor.

## Escopo
**Dentro:**
- Criar item no vault via cliente Bitwarden
- Consultar tabela `ciphers` no Postgres (`docker compose exec postgres psql ...`) e mostrar que `data`, `name`, `notes` estão cifrados (base64 de blob AES)
- Grep no código-fonte por rotas que decodifiquem `Cipher` no servidor — confirmar ausência (ver `src/api/core/ciphers.rs`)
- Documentar `src/crypto.rs`: server-side crypto helpers só existem para envelope de chaves / password reset admin (que ainda é AGPL-clean; não decifra vault)

**Fora:**
- Auditoria de segurança formal
- Análise de side-channels

## Critérios de aceitação
- [ ] `SELECT * FROM ciphers` mostra apenas ciphertext (screenshot ou dump anonimizado no PRD)
- [ ] Grep `rg -n "decrypt|decipher" src/` catalogado — cada uso classificado como "não toca vault plaintext"
- [ ] Documento curto em `docs/` (ou seção deste PRD) resume evidências
- [ ] Confirmado que backup do Postgres (dump) contém somente ciphertext

## Dependências
POC-01

## Riscos & mitigações
- **Alguma rota admin expor plaintext** → ler `src/api/admin.rs` linha a linha; se houver, tratar como vulnerabilidade e reportar.
- **Log com plaintext** → grep em `src/mail.rs`, `src/api/*.rs`; verificar níveis de log.

## Referências
- `src/crypto.rs`, `src/api/core/ciphers.rs`, `src/api/admin.rs`
- Bitwarden crypto whitepaper
