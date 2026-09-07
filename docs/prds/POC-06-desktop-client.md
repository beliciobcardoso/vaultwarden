# POC-06 — Testar cliente desktop Bitwarden conectado ao Vaultwarden local

**Status:** todo · **Prioridade:** P1 · **Backlog:** [POC-06](../backlog.md)

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
- [ ] Cliente instalado no Linux dev
- [ ] Login SSO funcional
- [ ] Sync bidirecional (criar no desktop → aparece no web e vice-versa)
- [ ] Comportamento com stack derrubada e religada documentado

## Dependências
POC-01

## Riscos & mitigações
- **HTTPS obrigatório** → mesma mitigação do POC-05 (Caddy + mkcert).
- **WebSocket bloqueado** → validar `notifications.rs`, testar com `wss://` se HTTPS entrar.

## Referências
- `src/api/notifications.rs`
- POC-05 (mesmas questões de HTTPS local)
