# POC-06 — Testar cliente desktop Bitwarden conectado ao Vaultwarden local

**Status:** done (com limitação) · **Prioridade:** P1 · **Backlog:** [POC-06](../backlog.md)

## Contexto
Aplicativo desktop Bitwarden (Windows/Linux/macOS) precisa apontar para o servidor local e funcionar com SSO.

## Problema
Análoga ao POC-05, mas em cliente nativo. Pode ter comportamento diferente para: TLS self-signed, WebSocket, autofill via integração de sistema.

## Objetivo
Cliente desktop conectado ao servidor local, login SSO concluído, sync do vault funcionando.

## Escopo
**Dentro:**
- Instalar cliente desktop em pelo menos 1 SO (Linux, já que dev usa Ubuntu)
- Apontar para Server URL local
- Login SSO
- Criar/editar item e confirmar sync com web vault
- Documentar comportamentos específicos (WebSocket, notificações)

**Fora:**
- Testes em Windows/macOS (deixar para pós-PoC, exceto se dev tiver acesso)
- Empacotamento corporativo (MSI/DMG/AppImage assinado)

## Critérios de aceitação
- [x] Cliente instalado no Linux dev (snap e flatpak testados)
- [x] Server URL customizada aceita (`https://pasty-lionesque-leerily.ngrok-free.dev`)
- [x] Login SSO concluído server-side (logs mostram `client_id=desktop` + `oidc-signin` 307 + `POST /identity/connect/token` 200 OK + `User ... logged in successfully`)
- [ ] Sync bidirecional — **não validado** (trave no unlock, ver "Limitação")
- [ ] Comportamento com stack derrubada e religada — **não avaliado** por causa da limitação
- [x] Limitação documentada + workaround definido

## Dependências
POC-01, INF-07

## Riscos & mitigações
- **HTTPS obrigatório** → resolvido via ngrok (INF-07).
- **WebSocket bloqueado** → não é o gargalo; WS conectou (`Accepting Rocket WS connection` no log).

## Limitação encontrada (2026-09-07)

**Sintoma:** após submeter master password na tela "Boas-vindas de volta" (ou "Seu cofre está bloqueado" em unlock subsequente), botão vira spinner indefinido. Vault nunca abre no cliente. Reproduzido em duas distribuições independentes:

- Snap `bitwarden` (canonical)
- Flatpak `com.bitwarden.desktop`

**Server-side não é o problema:**
- Fluxo SSO chega ao fim (`client_id=desktop`, callback `oidc-signin` 307, token endpoint 200 OK, `User belicio.cardoso@novacorrente.ind.br logged in successfully`).
- Refresh token cycle roda normalmente a cada ~5min (`POST /identity/connect/token` 200 OK).
- WebSocket estabelece (`Accepting Rocket WS connection from 172.27.0.1`).

**Cliente trava client-side** no processamento após receber o token e/ou durante derivação KDF / sync inicial. Sem tempo/interesse de debugar Electron (não é escopo da PoC).

## Workaround adotado

- **Uso corporativo:** web vault (via ngrok/HTTPS) + **extensão de navegador** (POC-05, validada).
- Extensão cobre: leitura, criação, edição, autofill em sites, geração de senhas.
- Web vault cobre: administração, gestão de organizações, emergency access, sends.

**Cliente desktop nativo não entra no escopo do rollout inicial.** Reavaliar em release futura do Bitwarden desktop, ou considerar clientes alternativos compatíveis com Bitwarden API (ex.: `rbw`, `goldwarden`) se demanda por CLI/nativo surgir.

## Conclusão

Autenticação SSO desktop **funciona no servidor**. Client-side desktop **não abre o vault** em nenhuma distribuição testada. Workaround via web + extensão atende os casos de uso corporativos da PoC. Desktop nativo fica fora do escopo do rollout.

## Referências
- `src/api/notifications.rs` (WebSocket)
- `src/api/identity.rs` (fluxo SSO + token)
- POC-05 (extensão — workaround adotado)
- INF-07 (HTTPS via ngrok — pré-requisito)
- Frames de evidência (temporários, não versionados): `/tmp/poc06-frames/`, `/tmp/poc06-video1/`
