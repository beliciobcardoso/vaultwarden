# POC-05 — Testar extensão de navegador Bitwarden conectada ao Vaultwarden local

**Status:** done · **Prioridade:** P1 · **Backlog:** [POC-05](../backlog.md)

## Contexto
Extensão oficial do Bitwarden (Chrome/Firefox/Edge) precisa aceitar `Self-hosted → Server URL = http://localhost:8080` (ou HTTPS reverso, ver observação abaixo) e completar login SSO.

## Problema
Fluxo SSO na extensão passa por popup / redirect que depende de HTTPS válido em muitos cenários. Precisa-se descobrir se `http://localhost:8080` funciona ou se precisa de reverse proxy HTTPS local (Caddy, mkcert).

## Objetivo
Extensão instalada, apontada para o servidor local, login SSO concluído, autofill funcionando em uma página de teste.

## Escopo
**Dentro:**
- Instalar extensão em Chrome/Firefox de teste
- Configurar Server URL (settings da extensão)
- Login via SSO
- Testar autofill em uma página de login qualquer (ex.: `httpbin.org/forms/post`)
- Registrar limitações (ex.: se popup SSO precisa HTTPS, decidir se adicionamos Caddy/mkcert ao compose)

**Fora:**
- Publicar extensão custom (não é o modelo — usamos a oficial)
- Configuração empresarial via GPO (produção)

## Critérios de aceitação
- [x] Extensão conectada ao servidor local
- [x] Login SSO completo na extensão
- [x] Autofill funcional
- [x] Documentado se HTTPS local é necessário; se sim, PRD novo para "reverse proxy HTTPS local"

## Dependências
POC-01

## Riscos & mitigações
- **Popup SSO recusar `http://`** → adicionar Caddy + mkcert; abrir PRD dedicado.
- **CORS ou CSP quebrar** → checar logs Vaultwarden e devtools da extensão.

## Notas de implementação (2026-09-07)

- Server URL customizada: `https://pasty-lionesque-leerily.ngrok-free.dev` (INF-07). `http://localhost:8080` **não funciona** — extensão exige HTTPS no fluxo SSO.
- Fluxo validado: login SSO Azure AD via extensão + master password unlock + autofill em página de teste.
- Em produção: substituir ngrok por domínio próprio + cert público (PRD futuro, fora do escopo PoC).

## Referências
- `src/api/notifications.rs` (WebSocket para sync)
- INF-07 (HTTPS ngrok — pré-requisito do fluxo)
- Docs Bitwarden self-hosted setup extension
