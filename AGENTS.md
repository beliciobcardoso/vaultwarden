# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) and other AI coding agents when working with code in this repository. This is the primary agent instruction file for this project — prefer it over any CLAUDE.md.

## Project Context

Fork of [dani-garcia/vaultwarden](https://github.com/dani-garcia/vaultwarden) maintained for **Nova Corrente** as an internal corporate password vault (project codename: **NewChainSafe**).

- Fork remote: `origin` → `github.com/beliciobcardoso/vaultwarden`
- Upstream remote: `upstream` → `github.com/dani-garcia/vaultwarden`
- Active branch for all internal work: `developer` (all PoC/customization commits go here, never on `main`)
- `main` mirrors upstream; keep it clean to allow rebases / merges from upstream.
- Internal product docs live in `docs/` (versioned in the fork by explicit user decision): `overview.md`, `backlog.md`, `market-analysis.md`. **Read `docs/backlog.md` first** — it defines current priorities (INF/POC/BUG/FIX IDs) and the hard constraint: *no production deploy until PoC concluded and validated locally*.

Target stack for PoC: Vaultwarden (Rust) + PostgreSQL + Azure AD OIDC (standard SSO mode — **do NOT enable `SSO_ONLY`**, tracked as BUG-01, redirect-loop against Azure AD).

## Build & Run

Cargo requires **at least one DB feature** to be enabled — the default feature set is empty on purpose.

```bash
# Build (pick one backend — postgresql is the target for this project)
cargo build --features postgresql --release

# Dev build with sqlite (fastest local iteration, no external DB)
cargo build --features sqlite

# Run locally
cargo run --features postgresql

# Format / lint / typos (mirror what CI runs — see .github/workflows/build.yml)
cargo fmt --all -- --check
cargo clippy --features postgresql,enable_mimalloc -- -D warnings
typos          # spellcheck configured via .typos.toml
```

DB features are mutually exclusive at runtime for schema generation; Diesel picks migrations from `migrations/{sqlite,mysql,postgresql}/` matching the enabled feature.

## Tests

```bash
# Rust unit/integration tests (compiled per DB backend)
cargo test --features sqlite

# Playwright E2E — spins Vaultwarden + DBs + maildev via docker compose
cd playwright
npm ci
cp .env.template .env       # first time only
npx playwright test                          # full suite
npx playwright test tests/sso_login.spec.ts  # single spec
npx playwright test -g "SSO"                 # grep by title
npx playwright test --headed --debug         # interactive debug
```

`playwright/docker-compose.yml` is the reference stack (Vaultwarden + PostgreSQL + MySQL + MailDev + Playwright runner). Use it as the starting point for INF-02 (local PoC compose) — do not re-invent it.

Pre-commit hooks live in `.pre-commit-config.yaml` (fmt, clippy, typos, hadolint). Install with `pre-commit install` if working on this repo long-term.

## Architecture — Big Picture

Vaultwarden is a **single Rust binary** that reimplements the Bitwarden server API. It is stateless-ish (session/push state aside) and delegates persistence to Diesel + PostgreSQL/MySQL/SQLite. All Bitwarden official clients (web vault, browser extension, desktop, mobile) talk to it unchanged.

### Request lifecycle

`src/main.rs` boots Rocket, mounts routers from `src/api/`, and starts background jobs (email queue, WebSocket notifications, push relay). Auth middleware in `src/auth.rs` validates JWTs on every request; SSO flows are in `src/sso.rs` + `src/sso_client.rs` (OIDC discovery, token exchange, provider quirks — this is where Azure AD integration lives and where BUG-01 must be investigated).

### `src/api/` — HTTP surface (grouped by Bitwarden API area)

- `core/` — main Bitwarden endpoints. **Very large files** by design (single-file-per-resource): `ciphers.rs` (~76KB, vault items), `organizations.rs` (~110KB, orgs/collections/groups/policies), `accounts.rs` (~62KB, user lifecycle), `emergency_access.rs`, `sends.rs`, `events.rs`, plus `two_factor/` (authenticator, webauthn, duo, duo_oidc, yubikey, email).
- `identity.rs` — OAuth2/OIDC token endpoints (login, refresh, SSO callback wiring).
- `admin.rs` — the admin backend UI (feature-gated by `ADMIN_TOKEN`).
- `icons.rs` — website icon proxy/cache.
- `notifications.rs` — WebSocket + SignalR-compat hub for live vault sync to clients.
- `push.rs` — mobile push relay (Bitwarden push service).
- `web.rs` — serves the bundled web vault static assets.

### `src/db/`

- `schema.rs` — Diesel-generated table definitions.
- `models/` — Diesel structs + business methods (query builders, permission checks, tenant/org scoping live here — this is the layer to touch for anything data-related).
- `mod.rs` — connection pool, backend abstraction over sqlite/mysql/postgresql.
- `query_logger.rs` — dev-only SQL tracing.

### Cross-cutting

- `src/config.rs` (~88KB) — every runtime setting; env vars documented in `.env.template`. Read it before adding a new toggle.
- `src/auth.rs` — JWT issuing/validation, per-request identity guards (Rocket request guards).
- `src/crypto.rs` — server-side crypto helpers only. **Vault content is end-to-end encrypted client-side**; the server never sees plaintext. Do not add server-side decryption paths.
- `src/mail.rs` — Lettre-based mailer, all transactional templates in `src/static/templates/email/`.
- `src/http_client.rs` — outbound HTTP with a shared, hardened client (SSRF guards for icon/webhook fetches).
- `src/ratelimit.rs` — governor-based rate limiting on sensitive endpoints (login, 2FA, password hint).
- `src/storage.rs` — attachment/send file storage abstraction (local fs vs S3-compatible).
- `macros/` — internal proc-macro crate (workspace member); pulls small compile-time helpers out of the main crate.

### Migrations

Three parallel trees under `migrations/{sqlite,mysql,postgresql}/`. Any schema change requires **all three** to stay in lockstep — Diesel picks the tree matching the compiled feature. Migration files are timestamped and applied at startup.

### Docker & release

- `docker/Dockerfile.j2` is the source of truth; `Dockerfile.debian` and `Dockerfile.alpine` are Jinja-rendered from it via `docker/render_template`. Edit the `.j2`, then re-render — never hand-edit the two output Dockerfiles (CI enforces this in `.github/workflows/check-templates.yml`).
- Multi-arch builds are driven by `docker-bake.hcl` (`bake.sh`, `podman-bake.sh`).
- Release workflow: `.github/workflows/release.yml`.

## Regras de Operação (obrigatórias)

Estas regras são inegociáveis. Antes de qualquer ação, verifique se ela se enquadra:

1. **Proibido commitar sem ordem explícita do usuário.** `git commit` (e qualquer variação: `--amend`, `commit -a`) só executa após o usuário mandar. Ficar com mudanças staged/unstaged é o estado padrão até ordem contrária.
2. **Proibido criar Pull Requests sem ordem explícita do usuário.** `gh pr create` e equivalentes só rodam quando o usuário pedir. Push para branch remota também precisa de ordem — não presuma.
3. **Toda nova tarefa começa em branch criada a partir de `developer`.** Nunca trabalhar direto em `developer` ou `main`. Nome de branch: `<tipo>/<id-do-prd>-<slug-curto>` (ex.: `feat/INF-02-docker-compose`, `fix/BUG-01-sso-redirect-loop`).
4. **Toda tarefa começa a partir de um PRD.** Antes de escrever código:
   - Verificar se existe PRD para a tarefa em `docs/prds/` (arquivo por tarefa: `docs/prds/<ID>-<slug>.md`).
   - Se **não existir**, **criar o PRD primeiro** — problema, objetivo, escopo, critérios de aceitação, riscos — e só depois iniciar a implementação.
   - PRD é a fonte da verdade da tarefa; escopo fora do PRD não entra sem atualizar o PRD antes.
5. **Toda tarefa concluída atualiza o PRD correspondente** marcando os itens de aceitação como concluídos (checkbox `[x]`) e, se aplicável, atualiza o status no `docs/backlog.md` (`todo` → `done`). Sem essa atualização a tarefa não é considerada concluída.

## Working on This Fork

1. Always branch **from `developer`**, not `main`. PRs internal to the fork target `developer`.
2. Before proposing any upstream contribution, isolate the change on a topic branch off `main` (or off the current upstream tag) so it can be sent to `dani-garcia/vaultwarden` without Nova Corrente-specific baggage. `docs/` and any Nova Corrente config must not appear in upstream PRs.
3. When touching Azure AD / SSO code, coordinate with BUG-01 investigation in `docs/backlog.md` (FIX-01). Reproduce the redirect loop before proposing a fix.
4. Schema changes require migrations in **all three** backend trees. Compile with each feature (`--features sqlite`, `--features mysql`, `--features postgresql`) to verify.
5. Vault content is E2E-encrypted. Any feature that appears to need server-side plaintext access is almost certainly wrong — verify against the Bitwarden crypto model first.
