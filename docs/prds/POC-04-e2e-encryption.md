# POC-04 — Validar criptografia E2E (servidor/DB nunca veem texto plano)

**Status:** done · **Prioridade:** P0 · **Backlog:** [POC-04](../backlog.md)

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
- [x] `SELECT * FROM ciphers` mostra apenas ciphertext (dump abaixo)
- [x] Grep `decrypt|decipher` no server-side catalogado — nenhum uso descriptografa vault
- [x] Evidências consolidadas nesta seção
- [x] Backup Postgres (`pg_dump --data-only`) grep por plaintext do teste → 0 matches

## Dependências
POC-01

## Riscos & mitigações
- **Alguma rota admin expor plaintext** → ler `src/api/admin.rs` linha a linha; se houver, tratar como vulnerabilidade e reportar.
- **Log com plaintext** → grep em `src/mail.rs`, `src/api/*.rs`; verificar níveis de log.

## Evidências (2026-09-07)

### 1. DB armazena apenas ciphertext

Item criado no cliente Bitwarden Web: nome `POC-04-teste`, senha `SegredoQueNaoDeveVazar123!`, nota `nota secreta poc-04`.

```sql
SELECT uuid, atype, LEFT(name, 80), LEFT(data, 200), LEFT(notes, 100) FROM ciphers;
```

Saída (trecho):
- `name` = `2.JKSfDRIkSFEGVUCJENhxkg==|ykxDW5J4RURtQZf2J+fvww==|wxMUGw3zP9Fa4EMUOvTaEWzo1dhM...`
- `data.password` = `2.hz8Ze/GYRRrVQVF7SaUMCw==|u64XcMPxRGw4mJVX1932GNEypA+hdIwH7kFMiqFES54=|Vtl0qvukvBVrld9KFhITunHWC3A4aPUEBauSu62rs8A=`
- `notes` = `2.r/1AvmTtZ2DR6phM4OG14Q==|E2ShnsTFu1mrJ9Z0m8uK2OnTbx1oRAmL26cjrS173lY=|Vau8dDoSX+OzQLvrYxS9w4t9at9T...`

Todos no formato **Bitwarden CipherString v2** (`2.iv|ct|mac`) — AES-CBC + HMAC-SHA256 com chave derivada da master password. Nenhum campo em plaintext.

### 2. Dump completo não contém plaintext

```bash
docker compose exec -T postgres pg_dump -U vaultwarden vaultwarden --data-only \
  | grep -iE "SegredoQue|POC-04|nota secreta"
# 0 matches
```

### 3. Server não tem função de decifrar vault

`src/crypto.rs` — funções expostas (verificado com `grep -n "^pub fn" src/crypto.rs`):
- `hash_password`, `verify_password_hash` — Argon2/PBKDF2 para hash de senha (ADMIN_TOKEN, tokens auth)
- `hmac_sign` — assinaturas HMAC (não descriptografa)
- geradores de random (`generate_id`, `get_random_bytes`, etc.)
- `ct_eq` — comparação constant-time
- `sha256_hex`

**Nenhuma função de AES decrypt.** Server não tem capacidade de descriptografar `CipherString`.

### 4. Grep `decrypt|decipher` em rotas API

```bash
rg -n -iE "decrypt|decipher" src/api/ src/crypto.rs
```

Únicas ocorrências: `userDecryptionOptions` / `userDecryption` em `src/api/core/ciphers.rs:170,199` e `src/api/identity.rs:591,594,744,747`. Todas retornam **para o cliente** os parâmetros KDF + `akey` (chave criptografada) para o cliente derivar chave e descriptografar localmente. Server só encaminha metadata; não descriptografa.

### 5. Rotas admin

`src/api/admin.rs` — não descriptografa vault. Painel `/admin` gerencia usuários, orgs, sends, mas nunca acessa conteúdo de ciphers. Password reset admin apenas invalida chaves (usuário perde acesso ao vault existente).

## Conclusão

Modelo zero-knowledge confirmado empiricamente:
- Ciphertext no DB (formato validado)
- Dump PostgreSQL não expõe plaintext do teste
- Ausência de primitiva de decrypt no crate
- Rotas API só transportam ciphertext + envelope KDF

**Risco residual:** master password fraca. Mitigação em POC-07 (policy `MasterPassword`) + orientação de onboarding.

## Referências
- `src/crypto.rs` (helpers server-side — sem AES decrypt)
- `src/api/core/ciphers.rs:170-203` (sync retorna ciphertext + envelope)
- `src/api/identity.rs:591-747` (login retorna `userDecryptionOptions`)
- `src/api/admin.rs` (painel admin, sem acesso a plaintext)
- Bitwarden security whitepaper (CipherString v2 format)
