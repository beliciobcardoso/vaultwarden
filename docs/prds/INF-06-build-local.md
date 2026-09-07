# INF-06 — Build local do fork

**Status:** todo · **Prioridade:** P0 · **Backlog:** [INF-06](../backlog.md)

## Contexto
Decisão de negócio da Nova Corrente: **o binário em produção deve vir do fork versionado** (`github.com/beliciobcardoso/vaultwarden`), não da imagem oficial `vaultwarden/server`. Motivo: além da correção de bugs (BUG-01/FIX-01), o time pretende aplicar melhorias no projeto — permitido pela AGPL v3 (ver seção "Obrigações AGPL v3" em `AGENTS.md`).

INF-02 subiu o PoC com a imagem oficial (`vaultwarden/server:1.34.3`) por atrito mínimo. Este PRD substitui isso: o `compose.yaml` passa a construir a imagem a partir de `docker/Dockerfile.debian` do fork.

## Problema
Imagem oficial não reflete nossos patches, não permite rastrear em prod o commit exato que gerou o binário, e não permite aplicar melhorias custom. Auditoria e correção de bugs ficam impraticáveis sem controle da build.

## Objetivo
`docker compose up -d --build` constrói a imagem localmente a partir do fork (`docker/Dockerfile.debian`), com tag versionada da Nova Corrente (ex.: `newchainsafe/vaultwarden:1.34.3-nc.1`), rodando na PoC e pronta para prod.

## Escopo

**Dentro:**
- Substituir bloco `image:` do serviço `vaultwarden` no `compose.yaml` por `build:` apontando para `docker/Dockerfile.debian`
- Definir esquema de versionamento: `<upstream-tag>-nc.<n>` (ex.: `1.34.3-nc.1`, `1.34.3-nc.2`, `1.35.0-nc.1`)
- Passar `--features postgresql,enable_mimalloc` ao build (backend PoC/prod)
- Adicionar `image: newchainsafe/vaultwarden:<tag>` no serviço para nomear a imagem construída
- Documentar no PRD: fluxo de build, rebuild, cache, tamanho esperado, tempo esperado
- Confirmar que o `Dockerfile.debian` da branch `developer` é o mesmo que o upstream (nenhum patch de Dockerfile ainda) — se divergir no futuro, editar `docker/Dockerfile.j2` e re-renderizar via `docker/render_template` (nunca hand-editar `.debian`/`.alpine`)
- Atualizar `AGENTS.md` (seção "Build & Run") caso o comando canônico mude
- Manter compose funcional sem passar `--build` a cada vez (imagem em cache local)

**Fora (escopo de outras tarefas futuras):**
- Publicar imagem em registry interno (Harbor/ECR/ACR) — vira `INF-07` quando produção entrar em pauta (bloqueada pela restrição-mestra até PoC concluída)
- Pipeline CI que builda automaticamente por push em `developer`/tags — vira PRD dedicado
- Multi-arch (arm64) — não é requisito atual
- Imagem `alpine` — Debian é o alvo; alpine fica se surgir necessidade

## Critérios de aceitação
- [ ] `compose.yaml` usa `build:` no serviço `vaultwarden`, apontando `context: .` e `dockerfile: docker/Dockerfile.debian`
- [ ] `image:` nomeia a build local: `newchainsafe/vaultwarden:<tag>`
- [ ] Feature `postgresql,enable_mimalloc` habilitada no build
- [ ] `docker compose build vaultwarden` conclui sem erros
- [ ] `docker compose up -d` sobe a stack com a imagem local (validar via `docker inspect newchainsafe-vaultwarden --format '{{.Config.Image}}'`)
- [ ] `curl -sf http://localhost:8080/alive` responde (mesmo critério do INF-02)
- [ ] Tempo do primeiro build documentado (esperado 10-15 min); rebuilds incrementais documentados (esperado <2 min quando só código muda)
- [ ] Comando de rebuild documentado no PRD e/ou em `AGENTS.md`
- [ ] Esquema de tag `<upstream>-nc.<n>` documentado

## Dependências
- INF-02 (compose base — done)

## Riscos & mitigações
- **Build muito lento na primeira vez** → esperado; documentar; considerar cache mount BuildKit se virar problema. Cache local do Docker resolve rebuilds seguintes.
- **Falta de dependências no host** (rustc, cargo — não deveriam ser necessários porque build ocorre dentro do container do `Dockerfile.debian`, que instala tudo) → confirmar; se falhar, avaliar bind mounts ou installer.
- **Divergência acidental de `Dockerfile.debian` vs `Dockerfile.j2`** → CI upstream (`.github/workflows/check-templates.yml`) alerta; nós replicamos essa checagem localmente antes de qualquer edição de Dockerfile.
- **Tag/versão inconsistente entre `Cargo.toml`, git tag e imagem** → convenção `<upstream-tag>-nc.<n>` amarrada a tag git anotada no fork; documentar processo de bump.

## Referências
- `docker/Dockerfile.debian`, `docker/Dockerfile.j2`, `docker/render_template`
- `docker/docker-bake.hcl` (referência de como upstream orquestra build)
- `AGENTS.md` → seções "Build & Run" e "Obrigações AGPL v3"
- INF-02 (compose atual)
