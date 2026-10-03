---
description: Organiza mudanças em lotes lógicos com mensagens Conventional Commits num plano em docs/commits/ e, após uma aprovação única, commita todos os lotes sem interrupção
argument-hint: [opcional - foco/diretório para o agrupamento, ex "só o módulo auth"]
---

Você vai conduzir o fluxo completo de commits, no thread principal (o gate de aprovação exige conversa com o usuário — NÃO delegue este fluxo a um subagente).

Foco informado pelo usuário: $ARGUMENTS

## Retomada
Antes de tudo, procure em `docs/commits/` um `COMMITS-*.md` com lotes `pendente` ou `falhou: ...`. Se houver, pergunte: retomar esse plano (pula direto para a Fase 4, a partir do primeiro lote não commitado) ou descartá-lo e gerar um novo.

## Fluxo

1. **Agrupar sem stagear** — siga a skill `stage-logical-commits` (.claude/skills/stage-logical-commits/SKILL.md) em **modo plano**: levante o estado, aplique as exclusões e agrupe TODOS os lotes em vertical slices (em projeto SDD, um lote por estágio do plan.md, com o lote `docs(sdd)` de plan.md/manifest.md como último). NÃO execute `git add` nesta fase. Se já houver algo staged ao iniciar, pergunte o que fazer antes de qualquer coisa.

2. **Gerar as mensagens** — siga a skill `conventional-commits` (.claude/skills/conventional-commits/SKILL.md) em **modo plano**, uma mensagem por lote, a partir de `git diff -- <arquivos do lote>` (untracked: leia o arquivo do disco).

3. **Gravar o plano** — leia `.claude/skills/stage-logical-commits/references/commit-plan-template.md` e grave `docs/commits/COMMITS-AAAAMMDD-HHMM.md` (crie a pasta se preciso) com todos os lotes em ordem de dependência, todos com status `pendente`, mais exclusões e alertas.
   → **GATE (único)**: informe o caminho do arquivo e um resumo curto (número de lotes + subject de cada um). Diga que o usuário pode editar o arquivo (mover arquivos, ajustar mensagens, remover lotes) antes de aprovar. AGUARDE o "ok". Ajustes pedidos no chat = atualize o arquivo e apresente de novo. NUNCA execute `git add` ou `git commit` antes deste "ok".

4. **Executar sem interrupção** — após o "ok", RELEIA o arquivo (ele é a fonte da verdade, inclusive com as edições do usuário) e, para cada lote `pendente`, em ordem, sem pedir nova confirmação:
   1. `git add` arquivo a arquivo, apenas os listados no lote;
   2. confira com `git diff --staged --name-only` que o staged é exatamente a lista do lote;
   3. `git commit` com um `-m` por parágrafo da mensagem aprovada (subject, body, trailers), preservando-a literalmente;
   4. pegue o hash (`git log -1 --format=%h`) e atualize o lote no arquivo: `commitado — <hash>`, mais a linha no log de execução.
   - Hook pre-commit que MODIFICA arquivos (formatadores): re-stageie só os arquivos do lote, commite de novo, anote no log e siga.
   - **Pare a execução** somente se: o hook falhar sem correção trivial, o staged divergir da lista, ou um arquivo listado sumir/não tiver mais mudanças. Marque o lote como `falhou: <motivo>`, deixe os seguintes como `pendente`, desfaça o stage do lote falho (`git restore --staged <arquivos>`) e devolva ao usuário com a saída do erro. Um novo `/commit` retoma o plano.

5. **Resumo** — preencha a seção final do arquivo e mostre no chat: commits criados (hash + subject), o que ficou de fora pelas exclusões e o caminho do plano, que fica como registro.

## Regras
- NUNCA `git push` — fora do escopo deste fluxo; apenas mencione ao final que os commits são locais.
- NUNCA `git add .` / `-A`, nem commit --amend ou rebase, salvo pedido explícito.
- O plano em `docs/commits/` nunca entra em commit (`docs/` está nas exclusões do stager).
- Mensagens são commitadas exatamente como estão no arquivo aprovado; não as "melhore" durante a execução.
- Depois do "ok", não peça novas aprovações: as únicas interrupções são as falhas listadas no passo 4.
