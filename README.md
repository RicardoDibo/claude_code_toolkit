# SDD Kit (completo) — Spec Driven Development para Claude Code

*[Read this in English](#sdd-kit-full--spec-driven-development-for-claude-code)*

## Instalação
Copie o conteúdo para a raiz do projeto (merge com o .claude/ existente —
seus settings.json e agents atuais não são tocados):

```
seu-projeto/
└── .claude/
    ├── commands/  sdd-new, sdd-feature, debug,           ← pontos de entrada (/)
    │              code-improve, code-audit, sonar-fix, commit
    ├── agents/    feature-implementer, contract-auditor,
    │              code-improver, audit-recon (Sonnet),
    │              security-hunter, finding-verifier,
    │              sonar-triager (Sonnet), sonar-fixer (Haiku)
    │              (+ seu db-reader, já existente)
    └── skills/
        ├── sdd-greenfield, sdd-brownfield               ← orquestradoras
        ├── prd-creator, sdd-feature-author,             ← especialistas SDD
        │   sdd-implementer
        ├── sdd-debug                                    ← debug por evidências
        ├── code-improve                                 ← revisão de código
        ├── code-audit                                   ← auditoria de segurança + qualidade
        ├── sonar-fix                                    ← correções do servidor SonarQube
        │   (references/webapi.md + vscode-extension.md)
        └── stage-logical-commits, conventional-commits  ← fluxo de commits
```

## Dependências
As Provas de dados delegam ao agente `db-reader` (seu, em
.claude/agents/). Sem ele, essa fase degrada para execução manual.

A fase de Qualidade usa o agente `code-improver`, incluído no kit. Ele
revisa no mesmo molde do /code-review: caça bugs de correção primeiro e,
depois, reúso, simplificação, eficiência, altitude e cobertura de testes.
Cada achado leva cenário de falha, veredito (CONFIRMED ou PLAUSIBLE) e
severidade, e o nível (low a xhigh) define quantos achados entram.
O mesmo agente roda fora do pipeline pelo /code-improve.

O /code-audit audita o repositório inteiro (ou um diff, branch ou
caminho) com foco em segurança. O `audit-recon` (Sonnet) mapeia
arquitetura, entry points e fronteiras de confiança num coverage ledger;
o `security-hunter` roda os scanners e caça fonte→sink por classe de
ataque; o `finding-verifier`, que nunca é quem achou, tenta refutar cada
candidato. O passe de qualidade é do `code-improver`. Scanners são
opcionais e usados só se instalados: gitleaks/trufflehog (segredos),
semgrep/opengrep, bandit, gosec, brakeman (SAST), osv-scanner, trivy,
grype, npm audit, pip-audit, govulncheck, cargo audit (dependências),
trivy config, checkov, hadolint (IaC) e CodeQL (perfil deep). Sem eles,
a auditoria cai para Grep/Glob/Read. Nada é instalado; os comandos que
usam rede são mostrados e aprovados antes. O settings.json do kit não
libera scanners: cada comando pede permissão, ou você os libera no
settings.local.json do seu projeto.

O /sonar-fix puxa os achados do **servidor SonarQube** pela Web API —
Community Build, Server ou Cloud, inclusive um container local em
http://localhost:9000. A Fase 0 pergunta URL, token e project key a cada
sessão (nada de conexão gravada no projeto; o token fica num config do
curl no scratchpad, nunca na linha de comando) e sonda a instância:
versão, esquema de autenticação, projeto, contagens, quality gate e os
nomes de parâmetro válidos naquela versão, via /api/webservices/list —
é assim que a skill funciona em qualquer versão sem parâmetro chumbado.
A extensão **SonarQube for IDE** do VS Code fica como complemento, para
verificar localmente se o achado sumiu entre lotes.
O trabalho é dividido entre dois agentes: o `sonar-triager` (Sonnet)
pagina as issues, lê a descrição de cada regra na própria instância
(/api/rules/show) e grava o plano de lotes em
docs/sonarqube/LOTES-*.md com os issue keys, e o `sonar-fixer` (Haiku)
aplica um lote por vez, verificando localmente com testes, lint e
build. Lotes de risco alto ou que o Haiku
não consegue resolver são refeitos com Sonnet. No fim, a skill oferece o
rescan (sonar-scanner, mvn, gradle ou container, com o comando aprovado
antes) para confirmar contra o quality gate. A skill **não escreve no
servidor**: falsos positivos e hotspots saem como comandos prontos no
relatório, para você executar.

## Comandos
| Comando | Uso |
|---------|-----|
| /sdd-new descrição      | software novo (greenfield): PRD → waves → implementação |
| /sdd-feature descrição  | feature em sistema existente (brownfield): Discovery primeiro |
| /debug descrição+prints | sessão de debug orientada a evidências |
| /code-improve [nível] [alvo] [--fix] | revisão no molde do /code-review (diff, branch, F0X ou caminho); com --fix, aplica os achados aprovados |
| /code-audit [perfil] [alvo] [--fix] | auditoria de segurança + qualidade do repositório (quick, standard ou deep), com scanners instalados e verificação independente; com --fix, aplica os achados aprovados |
| /sonar-fix [conexão/foco] | conecta ao servidor SonarQube e corrige os achados do projeto, lote a lote por regra |
| /commit [foco]          | lotes lógicos + Conventional Commits num plano em docs/commits/, aprovado uma vez e executado sem interrupções |

## Integrações entre as peças
- Estágio do plan.md concluído = lote natural de commit → /commit
- feature-implementer NUNCA commita; reporta "lotes prontos para commit"
- code-improver grava features/F0X-*/quality.md; você decide cada achado
  (aplicar | dispensar), o feature-implementer aplica só os aprovados e
  preenche o Resultado → /commit (fix, perf, test ou refactor). No
  brownfield, o modo discovery faz o raio-X da vizinhança no discovery.md
- /code-improve: relatório em quality/QUALITY-*.md (ou no quality.md da
  feature); sem --fix só revisa, com --fix você escolhe os achados numa
  resposta e o feature-implementer aplica → /commit
- /code-audit: tudo em audit/<data>-<escopo>/ (architecture.md,
  coverage-ledger.md, candidatos, raw/ e AUDIT.md); gate antes de rodar
  os scanners (comandos e rede aprovados) e, com --fix, gate para escolher
  os achados → feature-implementer → /commit (fix; atualizações de
  dependência viram comandos para você rodar, commitadas como build(deps));
  a qualidade sai em quality/QUALITY-*.md
- Bug no meio do pipeline → /debug; correção roteada (código | spec |
  dado | integração) e commitada como fix referenciando debug/BUG-*.md
- /sonar-fix: Gate 0 confirma a conexão (versão, projeto, quality gate,
  data da última análise); cada lote verde (testes, lint e build locais)
  → /commit; no fim, rescan opcional aprovado e relatório em
  docs/sonarqube/SONAR-*.md com contagens antes/depois e os comandos de
  write-back dos falsos positivos, para você rodar
- /commit grava todos os lotes e mensagens em docs/commits/COMMITS-*.md;
  você revisa ou edita o arquivo, aprova uma vez e todos os commits saem
  em sequência; o arquivo fica como registro dos hashes
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
    │              code-improve, code-audit, sonar-fix, commit
    ├── agents/    feature-implementer, contract-auditor,
    │              code-improver, audit-recon (Sonnet),
    │              security-hunter, finding-verifier,
    │              sonar-triager (Sonnet), sonar-fixer (Haiku)
    │              (+ your existing db-reader)
    └── skills/
        ├── sdd-greenfield, sdd-brownfield               ← orchestrators
        ├── prd-creator, sdd-feature-author,             ← SDD specialists
        │   sdd-implementer
        ├── sdd-debug                                    ← evidence-based debug
        ├── code-improve                                 ← code review
        ├── code-audit                                   ← security + quality audit
        ├── sonar-fix                                    ← SonarQube server fixes
        │   (references/webapi.md + vscode-extension.md)
        └── stage-logical-commits, conventional-commits  ← commit workflow
```

## Dependencies
The Data Proof phase delegates to the `db-reader` agent (yours, in
.claude/agents/). Without it, that phase degrades to manual execution.

The Quality phase uses the `code-improver` agent, included in the kit. It
reviews the same way /code-review does: correctness bugs first, then
reuse, simplification, efficiency, altitude and test coverage. Every
finding carries a failure scenario, a verdict (CONFIRMED or PLAUSIBLE)
and a severity, and the level (low to xhigh) sets how many findings make
the cut. The same agent runs outside the pipeline through /code-improve.

/code-audit audits the whole repository (or a diff, branch or path) with
a security focus. `audit-recon` (Sonnet) maps the architecture, entry
points and trust boundaries into a coverage ledger; `security-hunter`
runs the scanners and hunts source→sink per attack class;
`finding-verifier`, never the agent that found the issue, tries to
refute each candidate. The quality pass belongs to `code-improver`.
Scanners are optional and used only if installed: gitleaks/trufflehog
(secrets), semgrep/opengrep, bandit, gosec, brakeman (SAST), osv-scanner,
trivy, grype, npm audit, pip-audit, govulncheck, cargo audit
(dependencies), trivy config, checkov, hadolint (IaC) and CodeQL (deep
profile). Without them, the audit falls back to Grep/Glob/Read. Nothing
is installed; commands that use the network are shown and approved
first. The kit's settings.json does not allow any scanner: each command
asks for permission, or you allow them in your project's
settings.local.json.

/sonar-fix pulls findings from the **SonarQube server** through the Web
API — Community Build, Server or Cloud, including a local container at
http://localhost:9000. Phase 0 asks for URL, token and project key every
session (no connection config stored in the project; the token lives in a
curl config file in the scratchpad, never on a command line) and probes
the instance: version, auth scheme, project, counts, quality gate and the
parameter names valid on that version, via /api/webservices/list — that
is how the skill works on any version without hardcoded parameters.
The VS Code **SonarQube for IDE** extension stays as a complement, to
check locally that a finding is gone between batches.
The work is split between two agents: `sonar-triager` (Sonnet) pages
through the issues, reads each rule's description from the instance
itself (/api/rules/show) and writes the batch plan to
docs/sonarqube/LOTES-*.md with the issue keys, and `sonar-fixer`
(Haiku) applies one batch at a time, verifying locally with tests, lint
and build. High-risk batches, or
batches Haiku cannot resolve, are redone with Sonnet. At the end the
skill offers a rescan (sonar-scanner, mvn, gradle or container, with the
command approved first) to confirm against the quality gate. The skill
**never writes to the server**: false positives and hotspots come out as
ready-to-run commands in the report.

## Commands
| Command | Usage |
|---------|-------|
| /sdd-new description      | new software (greenfield): PRD → waves → implementation |
| /sdd-feature description  | feature in an existing system (brownfield): Discovery first |
| /debug description+screenshots | evidence-driven debug session |
| /code-improve [level] [target] [--fix] | review in the /code-review mold (diff, branch, F0X or path); with --fix, applies the approved findings |
| /code-audit [profile] [target] [--fix] | security + quality audit of the repository (quick, standard or deep), with installed scanners and independent verification; with --fix, applies the approved findings |
| /sonar-fix [connection/focus] | connects to the SonarQube server and fixes the project's findings, one batch per rule |
| /commit [focus]           | logical batches + Conventional Commits in a plan under docs/commits/, approved once and run without interruptions |

## Integration between the pieces
- A completed plan.md stage = natural commit batch → /commit
- feature-implementer NEVER commits; it reports "batches ready to commit"
- code-improver writes features/F0X-*/quality.md; you decide each finding
  (apply | dismiss), feature-implementer applies only the approved ones
  and fills in the Result → /commit (fix, perf, test or refactor). In
  brownfield, discovery mode X-rays the neighborhood into discovery.md
- /code-improve: report in quality/QUALITY-*.md (or in the feature's
  quality.md); without --fix it only reviews, with --fix you pick the
  findings in one reply and feature-implementer applies them → /commit
- /code-audit: everything in audit/<date>-<scope>/ (architecture.md,
  coverage-ledger.md, candidates, raw/ and AUDIT.md); a gate before the
  scanners run (commands and network approved) and, with --fix, a gate to
  pick the findings → feature-implementer → /commit (fix; dependency
  upgrades become commands for you to run, committed as build(deps));
  quality goes to quality/QUALITY-*.md
- Bug found mid-pipeline → /debug; the fix is routed (code | spec |
  data | integration) and committed as fix referencing debug/BUG-*.md
- /sonar-fix: Gate 0 confirms the connection (version, project, quality
  gate, last analysis date); each green batch (local tests, lint and
  build) → /commit; at the end, an approved optional rescan and a report
  in docs/sonarqube/SONAR-*.md with before/after counts and the write-back
  commands for the false positives, for you to run
- /commit writes every batch and message to docs/commits/COMMITS-*.md;
  you review or edit the file, approve once, and all commits run in
  sequence; the file stays as a record of the hashes
- manifest.md is the state machine: commands resume where they left off
