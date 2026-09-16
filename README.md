# SDD Kit (completo) — Spec Driven Development para Claude Code

*[Read this in English](#sdd-kit-full--spec-driven-development-for-claude-code)*

## Instalação
Copie o conteúdo para a raiz do projeto (merge com o .claude/ existente —
seus settings.json e agents atuais não são tocados):

```
seu-projeto/
└── .claude/
    ├── commands/  sdd-new, sdd-feature, debug,           ← pontos de entrada (/)
    │              sonar-fix, commit
    ├── agents/    feature-implementer, contract-auditor,
    │              sonar-triager (Sonnet), sonar-fixer (Haiku)
    │              (+ seus code-improver e db-reader, já existentes)
    └── skills/
        ├── sdd-greenfield, sdd-brownfield               ← orquestradoras
        ├── prd-creator, sdd-feature-author,             ← especialistas SDD
        │   sdd-implementer
        ├── sdd-debug                                    ← debug por evidências
        ├── sonar-fix                                    ← correções do SonarQube for IDE
        └── stage-logical-commits, conventional-commits  ← fluxo de commits
```

## Dependências
Fases de Qualidade e Provas de dados delegam aos agentes `code-improver`
e `db-reader` (seus, em .claude/agents/). Sem eles, essas fases degradam
para execução manual.

O /sonar-fix lê os achados da extensão **SonarQube for IDE** do VS Code
pela ferramenta de diagnósticos da IDE (Claude Code rodando no VS Code).
Sem essa integração, você cola os itens do painel Problems no chat.
O trabalho é dividido entre dois agentes: o `sonar-triager` (Sonnet) lê
os achados e grava o plano de lotes em sonar/LOTES-*.md, e o
`sonar-fixer` (Haiku) aplica um lote por vez. Lotes de risco alto ou que
o Haiku não consegue resolver são refeitos com Sonnet.

## Comandos
| Comando | Uso |
|---------|-----|
| /sdd-new descrição      | software novo (greenfield): PRD → waves → implementação |
| /sdd-feature descrição  | feature em sistema existente (brownfield): Discovery primeiro |
| /debug descrição+prints | sessão de debug orientada a evidências |
| /sonar-fix [foco]       | corrige os achados da extensão SonarQube for IDE, lote a lote por regra |
| /commit [foco]          | lotes lógicos + Conventional Commits com aprovação |

## Integrações entre as peças
- Estágio do plan.md concluído = lote natural de commit → /commit
- feature-implementer NUNCA commita; reporta "lotes prontos para commit"
- Bug no meio do pipeline → /debug; correção roteada (código | spec |
  dado | integração) e commitada como fix referenciando debug/BUG-*.md
- /sonar-fix: cada lote verde (testes + diagnósticos limpos) → /commit;
  relatório com falsos positivos em sonar/SONAR-*.md
- manifest.md é a máquina de estados: os comandos retomam de onde pararam

---

# SDD Kit (full) — Spec Driven Development for Claude Code

*[Leia isto em Português](#sdd-kit-completo--spec-driven-development-para-claude-code)*

## Installation
Copy the contents into your project root (merges with your existing
.claude/ — your current settings.json and agents are left untouched):

```
your-project/
└── .claude/
    ├── commands/  sdd-new, sdd-feature, debug,           ← entry points (/)
    │              sonar-fix, commit
    ├── agents/    feature-implementer, contract-auditor,
    │              sonar-triager (Sonnet), sonar-fixer (Haiku)
    │              (+ your existing code-improver and db-reader)
    └── skills/
        ├── sdd-greenfield, sdd-brownfield               ← orchestrators
        ├── prd-creator, sdd-feature-author,             ← SDD specialists
        │   sdd-implementer
        ├── sdd-debug                                    ← evidence-based debug
        ├── sonar-fix                                    ← SonarQube for IDE fixes
        └── stage-logical-commits, conventional-commits  ← commit workflow
```

## Dependencies
The Quality and Data Proof phases delegate to the `code-improver` and
`db-reader` agents (yours, in .claude/agents/). Without them, those
phases degrade to manual execution.

/sonar-fix reads findings from the VS Code **SonarQube for IDE** extension
through the IDE diagnostics tool (Claude Code running in VS Code).
Without that integration, you paste the Problems panel items into the chat.
The work is split between two agents: `sonar-triager` (Sonnet) reads the
findings and writes the batch plan to sonar/LOTES-*.md, and `sonar-fixer`
(Haiku) applies one batch at a time. High-risk batches, or batches Haiku
cannot resolve, are redone with Sonnet.

## Commands
| Command | Usage |
|---------|-------|
| /sdd-new description      | new software (greenfield): PRD → waves → implementation |
| /sdd-feature description  | feature in an existing system (brownfield): Discovery first |
| /debug description+screenshots | evidence-driven debug session |
| /sonar-fix [focus]        | fixes SonarQube for IDE findings, one batch per rule |
| /commit [focus]           | logical batches + Conventional Commits with approval |

## Integration between the pieces
- A completed plan.md stage = natural commit batch → /commit
- feature-implementer NEVER commits; it reports "batches ready to commit"
- Bug found mid-pipeline → /debug; the fix is routed (code | spec |
  data | integration) and committed as fix referencing debug/BUG-*.md
- /sonar-fix: each green batch (tests + clean diagnostics) → /commit;
  report with false positives in sonar/SONAR-*.md
- manifest.md is the state machine: commands resume where they left off
