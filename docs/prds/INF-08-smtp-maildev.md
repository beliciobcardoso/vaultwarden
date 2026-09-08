# INF-08 — Habilitar SMTP via MailDev (PoC local)

**Status:** done · **Prioridade:** P1 · **Backlog:** [INF-08](../backlog.md)

## Contexto
Convites de organização (POC-07), reset de senha, verificação de email e demais fluxos transacionais do Vaultwarden dependem de SMTP. Container `maildev` já está no `compose.yaml` (INF-02), com `SMTP_HOST=maildev`/`SMTP_PORT=1025` injetados no serviço vaultwarden, mas `MAIL_ENABLED` não estava setado — Vaultwarden default é mail off.

## Problema
Sem `MAIL_ENABLED=true`, `src/mail.rs` nem tenta enviar. POC-07 bloqueado: owner convida usuário → convite não sai → destinatário não aparece em "Precisa de confirmação".

## Objetivo
Habilitar envio de email transacional para PoC local, com todos os emails caindo no MailDev inbox web em `http://localhost:1080` (zero saída para internet, sem dependência de SPF/DKIM/tenant Azure).

## Escopo
**Dentro:**
- `MAIL_ENABLED=true` no `.env` e documentado em `.env.example`
- Restart do container vaultwarden com `--force-recreate` para relê env_file
- Validação end-to-end: convite de POC-07 chega no inbox MailDev

**Fora:**
- SMTP corporativo Microsoft 365 (fica para pré-prod: precisa SMTP AUTH habilitado no tenant, App Password ou OAuth2, SPF/DKIM alinhados)
- Templates customizados de email (padrão upstream é suficiente para PoC)
- Rate limiting específico de mail (default do governor já cobre)

## Critérios de aceitação
- [x] `MAIL_ENABLED=true` em `.env`
- [x] `.env.example` documenta a chave
- [x] Container reiniciado, `docker compose exec vaultwarden env | grep MAIL_ENABLED` mostra `true`
- [x] Convite de teste enviado → aparece em `http://localhost:1080` — assunto `Join Nova Corrente`, corpo com botão "Join Organization Now", From `NewChainSafe <naoresponda@novacorrente.ind.br>`
- [x] Link de aceite abre no web vault e permite o fluxo completo — validado pelo email `Invitation to Nova Corrente accepted` recebido em seguida no inbox do owner (suporte@)

## Evidências (2026-09-08)

Inbox MailDev (`http://localhost:1080`) mostra a sequência completa:

1. `Join Nova Corrente` → belicio.cardoso@novacorrente.ind.br — convite de organização enviado pelo Vaultwarden
2. `Invitation to Nova Corrente accepted` → suporte@novacorrente.ind.br — notificação para o owner de que belicio aceitou o convite

Além dos convites, o fluxo transacional geral está ativo — emails `New Device Logged In From Chrome/Linux` e `Welcome` chegam corretamente em ambas as caixas.

Nenhum email sai para a internet: MailDev captura tudo localmente. Ideal para PoC.

## Dependências
INF-02 (compose com serviço maildev)

## Riscos & mitigações
- **Vaultwarden não expor logs claros de falha SMTP** → aumentar `LOG_LEVEL=debug` se necessário durante troubleshooting.
- **Email de aceite conter URL interna (localhost) que não abre no navegador do destinatário** → em PoC os testes são no mesmo host; em prod resolver via `DOMAIN` público (INF-07 já cobre via ngrok).

## Referências
- `compose.yaml` (serviço `maildev` + injeção de `SMTP_*` no vaultwarden)
- `src/mail.rs`
- `src/config.rs` (chave `mail_enabled`)
- MailDev inbox: http://localhost:1080
