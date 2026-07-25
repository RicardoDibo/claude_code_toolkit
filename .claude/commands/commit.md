---
description: Organiza mudanças em lotes lógicos, gera mensagens Conventional Commits e commita com aprovação explícita, lote a lote
argument-hint: [opcional - foco/diretório para o agrupamento, ex "só o módulo auth"]
---

Você vai conduzir o fluxo completo de commits, no thread principal (os gates de aprovação exigem conversa com o usuário — NÃO delegue este fluxo a um subagente).

Foco informado pelo usuário: $ARGUMENTS

## Fluxo (repita até o working directory ficar limpo ou o usuário parar)

1. **Stagear um lote** — siga a skill `stage-logical-commits` (.claude/skills/stage-logical-commits/SKILL.md): levante o estado, aplique as exclusões, agrupe em vertical slices (em projeto SDD, um lote por estágio do plan.md) e stageie SOMENTE o primeiro lote, arquivo a arquivo.
   → **GATE 1**: apresente os arquivos staged + justificativa + prévia dos lotes restantes e AGUARDE a validação do usuário. Ele pode aprovar, mover arquivos entre lotes ou remover algo (`git restore --staged`). Não avance sem resposta.

2. **Gerar a mensagem** — siga a skill `conventional-commits` (.claude/skills/conventional-commits/SKILL.md) sobre o diff staged.
   → **GATE 2**: mostre a mensagem em bloco ```text e AGUARDE aprovação explícita. Ajustes pedidos = regenerar e apresentar de novo. NUNCA execute `git commit` sem o "ok" deste gate.

3. **Commitar** — `git commit` com a mensagem aprovada (use arquivo temporário ou -m múltiplos para preservar corpo e trailers).
   - Se um hook pre-commit falhar: mostre a saída, corrija se for trivial e autorizado, senão devolva ao usuário.
   - Se o hook MODIFICAR arquivos (formatadores): re-stageie apenas os arquivos já pertencentes ao lote e commite de novo; informe o ocorrido.
   - Confirme com `git log -1 --stat`.

4. **Loop** — se restam mudanças não commitadas, volte ao passo 1 para o próximo lote. Senão, resuma: commits criados (hash + subject) e o que ficou de fora pelas exclusões.

## Regras
- NUNCA `git push` — fora do escopo deste fluxo; apenas mencione ao final que os commits são locais.
- NUNCA `git add .` / `-A`, nem commit --amend ou rebase, salvo pedido explícito.
- Se já houver algo staged ao iniciar, pergunte o que fazer antes de qualquer coisa (é trabalho do usuário que você não vai atropelar).
- Em projeto SDD, após commitar lotes de código de uma feature, ofereça um lote final `docs(sdd)` com as atualizações de plan.md/manifest.md correspondentes.
