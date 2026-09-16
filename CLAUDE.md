# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este repositório

Um kit de extensões para o Claude Code, sem código de aplicação. Não há build, lint nem testes: o "produto" são arquivos Markdown em `.claude/` que o usuário copia (merge) para a raiz de outro projeto. Todo o conteúdo do kit é escrito em **português (pt-BR)**. `README.md` e `EXAMPLES.md` são bilíngues (seção PT seguida da seção EN, com links âncora entre elas); ao alterar um, atualize as duas seções.

## Arquitetura: três camadas

1. **Commands** (`.claude/commands/*.md`) são os pontos de entrada `/`. São finos: mandam ler a skill correspondente, repassam `$ARGUMENTS` e repetem os "lembretes invioláveis". A lógica fica na skill, não no command.
2. **Skills** (`.claude/skills/<nome>/SKILL.md`, com templates opcionais em `references/`) contêm o procedimento em fases com **gates** explícitos. Há dois tipos:
   - **orquestradoras** (`sdd-greenfield`, `sdd-brownfield`, `sonar-fix`): sequenciam fases e delegam o trabalho, sem executá-lo;
   - **especialistas** (`prd-creator`, `sdd-feature-author`, `sdd-implementer`, `sdd-debug`, `stage-logical-commits`, `conventional-commits`).
3. **Agents** (`.claude/agents/*.md`) são subagentes com `tools` e `model` fixados no frontmatter, e cada um tem um papel estreito: `feature-implementer`, `contract-auditor`, `sonar-triager` (Sonnet, só lê e planeja), `sonar-fixer` (Haiku, aplica um lote). As skills também delegam a `code-improver` e `db-reader`, que **não estão neste repo**: são do projeto do usuário, e sem eles essas fases viram execução manual.

| Command | Skill | Delega a |
|---------|-------|----------|
| `/sdd-new` | `sdd-greenfield` | prd-creator → sdd-feature-author → contract-auditor / feature-implementer |
| `/sdd-feature` | `sdd-brownfield` | igual ao anterior, com uma fase Discovery obrigatória (db-reader, code-improver) |
| `/debug` | `sdd-debug` | feature-implementer / sdd-feature-author, conforme a classificação da causa |
| `/sonar-fix` | `sonar-fix` | sonar-triager → sonar-fixer (com escalada para Sonnet) |
| `/commit` | `stage-logical-commits` + `conventional-commits` | sessão principal |

## Invariantes que atravessam as peças

Ao criar ou editar qualquer peça, preserve estes contratos entre elas:
- **Agentes nunca commitam.** Eles reportam "lotes prontos para commit", e o commit acontece só na sessão principal via `/commit`, lote a lote e com aprovação do usuário (nunca `git add .`, `--amend` ou `push`).
- **Artefatos em disco são a interface entre as fases e os agentes:**
  - SDD: `PRD.md`, `manifest.md` (a máquina de estados que permite retomar o pipeline) e `features/F0X-*/{contract,spec,plan}.md`;
  - debug: `debug/BUG-*.md`;
  - Sonar: `sonar/LOTES-*.md` (plano do triager para o fixer) e `sonar/SONAR-*.md` (relatório).
- **Hierarquia de autoridade no SDD:** `contract.md > spec.md > plan.md`. Uma mudança que quebra uma saída prometida exige aprovação do usuário.
- **Gates explícitos:** cada fase termina com um gate. Nenhuma correção ou implementação começa antes da aprovação correspondente (PRD, auditoria estática, plano de lotes).
- **Tipos de commit** ligados ao fluxo: `fix` para debug e bugs do Sonar, `refactor` para code smells, `docs(sdd)` como lote final das atualizações de plan/manifest.

## Convenções de escrita

- Frontmatter da skill: `name` e `description`. A `description` diz o que a skill faz **e quando usá-la**, listando gatilhos concretos, porque é por ela que a skill é acionada.
- Frontmatter do command: `description` e `argument-hint`. Frontmatter do agent: `name`, `description`, `tools` e `model`.
- Estilo: imperativo e denso, com fases numeradas, **Gate:** em negrito e uma seção final "Regras da sessão" ou "Proibições". Referências longas (templates, formatos) ficam em `references/`, e a skill manda lê-las antes de usar.
- Ao adicionar uma peça, atualize o README nas duas línguas: a árvore de instalação, a tabela de comandos e "Integrações entre as peças".
- `.claude/settings.json` só permite `git status`, `git add` e `git commit -m`.
