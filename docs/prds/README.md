# PRDs — NewChainSafe

Um PRD por tarefa do [backlog](../backlog.md). Nome do arquivo: `<ID>-<slug>.md`.

**Regra:** toda tarefa começa por um PRD. Se não existir, crie primeiro (ver `AGENTS.md` → Regras de Operação).

## Estrutura padrão de um PRD

- **Contexto** — de onde vem a demanda, referência ao item do backlog
- **Problema** — o que motiva
- **Objetivo** — o que a tarefa entrega
- **Escopo** — o que está dentro e o que está fora
- **Critérios de aceitação** — checkboxes verificáveis; toda tarefa concluída marca `[x]`
- **Dependências** — outros PRDs / decisões prévias
- **Riscos & mitigações**
- **Referências** — links para docs, issues, código

## Ciclo de vida

1. Criar PRD (`todo`).
2. Criar branch `<tipo>/<ID>-<slug>` a partir de `developer`.
3. Implementar conforme escopo.
4. Marcar critérios `[x]` conforme evolui.
5. Ao concluir: todos critérios `[x]` + atualizar status no `backlog.md` (`todo` → `done`).
