# INF-02 — Docker Compose: Vaultwarden + PostgreSQL

**Status:** done · **Prioridade:** P0 · **Backlog:** [INF-02](../backlog.md)

## Contexto
PoC exige stack local reprodutível. `playwright/docker-compose.yml` já provê referência (Vaultwarden + PG + MySQL + MailDev + runner Playwright).

## Problema
Sem compose PoC dedicado, cada dev sobe o serviço de forma diferente; validações de SSO, MFA e E2E ficam não-reprodutíveis.

## Objetivo
Um `compose.yaml` na raiz (ou `docker/compose.poc.yaml`) que sobe Vaultwarden + PostgreSQL + MailDev com um único `docker compose up`, pronto para receber a config de Azure AD (INF-05) e ser exercitado nos POC-0X.

## Escopo
**Dentro:**
- Serviço `vaultwarden` (imagem oficial ou build local via `docker/Dockerfile.debian`)
- Serviço `postgres` (16.x, volume persistente)
- Serviço `maildev` (SMTP local para reset/convite/2FA)
- Rede compose isolada
- Volumes persistentes para vault data e PG
- `.env` local (não versionado) alimenta variáveis
- Healthchecks

**Fora:** TLS termination, reverse proxy, backup, HA, imagem custom com patches — todos ficam para tarefas futuras.

## Critérios de aceitação
- [x] `docker compose up -d` sobe stack sem erros
- [x] Vaultwarden acessível em `http://localhost:8080` (`GET /alive` retorna timestamp)
- [x] PostgreSQL healthy; Vaultwarden conecta e roda migrations postgres (`__diesel_schema_migrations` + tabelas `ciphers`, `users`, `organizations`, etc. presentes)
- [x] MailDev acessível em `http://localhost:1080` (HTTP 200 na UI; recebimento efetivo de e-mail será confirmado em POC-01 quando houver usuário via SSO)
- [x] Volumes sobrevivem a `docker compose restart` e `docker compose down` (validado; dados persistem em `newchainsafe_postgres_data` e `newchainsafe_vaultwarden_data`)
- [x] `docker compose down -v` limpa tudo (usado no fluxo de troubleshoot durante INF-02)
- [x] Comandos documentados abaixo

## Comandos

```bash
# Subir stack
docker compose up -d

# Ver status
docker compose ps

# Logs
docker compose logs -f vaultwarden
docker compose logs -f postgres

# Restart (preserva volumes)
docker compose restart

# Parar (preserva volumes)
docker compose down

# Destruir tudo (APAGA VAULT DATA E DB — só em troubleshoot)
docker compose down -v

# Testes rápidos
curl -sf http://localhost:8080/alive          # timestamp = ok
curl -sf -o /dev/null -w "%{http_code}\n" http://localhost:1080/   # 200 = MailDev ok
docker exec newchainsafe-postgres psql -U vaultwarden -d vaultwarden -c "\dt"
```

## Notas de implementação

- **PostgreSQL 18** exige mount em `/var/lib/postgresql` (não `/var/lib/postgresql/data`) — o próprio container reclama e recusa iniciar se o mount estiver no path antigo. Compose já corrigido.
- **Porta 5432** frequentemente ocupada no host (Postgres local do dev). `.env.example` mantém `POSTGRES_PORT=5432`; se colidir, ajuste no `.env` local (ex.: `55432`).
- **`SSO_ONLY=false`** fixado no compose como salvaguarda contra BUG-01 (redirect loop). Não remover.
- **`ADMIN_TOKEN` deixado vazio** — admin backend desabilitado por default; INF-03 documenta geração segura quando/se for necessário habilitar.

## Dependências
- INF-01 (branch `developer`) — done
- INF-03 (variáveis de ambiente) — em paralelo, INF-03 alimenta INF-02

## Riscos & mitigações
- **Divergência com `playwright/docker-compose.yml`** → basear-se nele; não duplicar lógica.
- **Volume perdido em rebuild** → nomear volumes explicitamente, documentar comandos destrutivos.
- **Porta 5432/8080 já ocupada** → expor porta configurável via `.env`.

## Referências
- `playwright/docker-compose.yml`
- `docker/Dockerfile.debian`, `docker/start.sh`, `docker/healthcheck.sh`
- `.env.template` (raiz)
