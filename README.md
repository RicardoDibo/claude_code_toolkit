# SDD Kit (completo) — Spec Driven Development para Claude Code

*[Read this in English](#sdd-kit-full--spec-driven-development-for-claude-code)*

## Instalação
Copie o conteúdo para a raiz do projeto (merge com o .claude/ existente —
seus settings.json e agents atuais não são tocados):

```
seu-projeto/
└── .claude/
    ├── commands/  sdd-new, sdd-feature, debug, commit    ← pontos de entrada (/)
    ├── agents/    feature-implementer, contract-auditor
    │              (+ seus code-improver e db-reader, já existentes)
    └── skills/
        ├── sdd-greenfield, sdd-brownfield               ← orquestradoras
        ├── prd-creator, sdd-feature-author,             ← especialistas SDD
        │   sdd-implementer
        ├── sdd-debug                                    ← debug por evidências
        └── stage-logical-commits, conventional-commits  ← fluxo de commits
```

## Dependências
Fases de Qualidade e Provas de dados delegam aos agentes `code-improver`
e `db-reader` (seus, em .claude/agents/). Sem eles, essas fases degradam
para execução manual.

## Comandos
| Comando | Uso |
|---------|-----|
| /sdd-new descrição      | software novo (greenfield): PRD → waves → implementação |
| /sdd-feature descrição  | feature em sistema existente (brownfield): Discovery primeiro |
| /debug descrição+prints | sessão de debug orientada a evidências |
| /commit [foco]          | lotes lógicos + Conventional Commits com aprovação |

## Integrações entre as peças
- Estágio do plan.md concluído = lote natural de commit → /commit
- feature-implementer NUNCA commita; reporta "lotes prontos para commit"
- Bug no meio do pipeline → /debug; correção roteada (código | spec |
  dado | integração) e commitada como fix referenciando debug/BUG-*.md
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
    ├── commands/  sdd-new, sdd-feature, debug, commit    ← entry points (/)
    ├── agents/    feature-implementer, contract-auditor
    │              (+ your existing code-improver and db-reader)
    └── skills/
        ├── sdd-greenfield, sdd-brownfield               ← orchestrators
        ├── prd-creator, sdd-feature-author,             ← SDD specialists
        │   sdd-implementer
        ├── sdd-debug                                    ← evidence-based debug
        └── stage-logical-commits, conventional-commits  ← commit workflow
```

## Dependencies
The Quality and Data Proof phases delegate to the `code-improver` and
`db-reader` agents (yours, in .claude/agents/). Without them, those
phases degrade to manual execution.

## Commands
| Command | Usage |
|---------|-------|
| /sdd-new description      | new software (greenfield): PRD → waves → implementation |
| /sdd-feature description  | feature in an existing system (brownfield): Discovery first |
| /debug description+screenshots | evidence-driven debug session |
| /commit [focus]           | logical batches + Conventional Commits with approval |

## Integration between the pieces
- A completed plan.md stage = natural commit batch → /commit
- feature-implementer NEVER commits; it reports "batches ready to commit"
- Bug found mid-pipeline → /debug; the fix is routed (code | spec |
  data | integration) and committed as fix referencing debug/BUG-*.md
- manifest.md is the state machine: commands resume where they left off
