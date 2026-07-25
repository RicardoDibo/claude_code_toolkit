# SDD Kit (completo) — Spec Driven Development para Claude Code

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
