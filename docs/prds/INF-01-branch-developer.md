# INF-01 — Criar branch `developer` no fork

**Status:** done · **Prioridade:** P0 · **Backlog:** [INF-01](../backlog.md)

## Contexto
Todo trabalho interno da Nova Corrente deve ocorrer em uma branch isolada do `main`, que é reservado para espelhar o upstream `dani-garcia/vaultwarden`.

## Problema
Sem branch dedicada, customizações internas contaminariam `main` e impediriam rebase/merge limpo do upstream.

## Objetivo
Existir branch remota `developer` no fork `github.com/beliciobcardoso/vaultwarden`, base para todas as tarefas subsequentes.

## Escopo
**Dentro:** criar branch `developer` a partir do `main` do fork; garantir que ela é o alvo padrão de PRs internos.
**Fora:** proteção de branch (regras GitHub), pipeline CI dedicado.

## Critérios de aceitação
- [x] Branch `developer` existe em `origin` (fork)
- [x] `developer` foi criada a partir do `main` do fork
- [x] Documentação (`AGENTS.md`, `docs/backlog.md`) referencia `developer` como branch de trabalho

## Dependências
Nenhuma.

## Riscos & mitigações
- **Deriva vs. upstream** → manter `main` limpo e rebasear/merge periodicamente antes de sincronizar com `developer`.

## Referências
- `AGENTS.md` → seção "Working on This Fork"
- Fork: https://github.com/beliciobcardoso/vaultwarden
