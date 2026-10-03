# Exemplo de uso ponta a ponta (resumido)

*[Read this in English](#end-to-end-usage-example-summarized)*

## Dia 1 — greenfield
$ /sdd-new app de gestão de projetos com kanban para equipes pequenas

Fase 1: entrevista (rodadas curtas) → PRD com F01-auth, F02-projetos,
F03-kanban, F04-tarefas, F05-notificações; Matriz de Integração;
Waves: 0=[F01], 1=[F02], 2=[F03,F04], 3=[F05]. Você aprova → manifest.
Fase 2-3: artefatos da Wave 0 + auditoria estática APROVADA.
Fase 4: feature-implementer implementa F01 estágio a estágio.
→ relatório: "Lotes prontos para commit: estágio 1 (migrations),
estágio 2 (rotas), estágio 3 (middleware+verificação)".

$ /commit
Plano gravado em docs/commits/COMMITS-20261003-1430.md: 3 lotes, cada um
com arquivos, justificativa e mensagem pronta, por exemplo:
        lote 1: db/migrations/001_users.sql, src/models/user.ts
        feat(auth): add user table and model
        ...
        SDD-Feature: F01 / SDD-Stage: 1
GATE: você revisa o arquivo (pode editar lotes e mensagens) e diz ok
→ 3 commits em sequência, sem interrupção → arquivo atualizado com os hashes.

Fases 5-6: code-improver (2 achados, você aprova 1) → auditoria de
integração APROVADA → manifest ✔ → /commit lote docs(sdd) → Wave 1.

## Dia 12 — bug com print
$ /debug ao concluir tarefa o card some do kanban [print anexado]

Fase 0: transcrição do print ("PATCH /api/tasks/9f2 → 200; card ausente
na coluna Done"); teoria do usuário vs contract F04: o contrato diz que
task.completed é emitido — spec F03 diz que o board escuta board.updated.
Fase 4: classificação = INTEGRAÇÃO (gap na Matriz: evento errado).
Fase 5: sdd-feature-author corrige contratos F03/F04 + cenário novo →
feature-implementer aplica → regressão verde.
$ /commit  → fix(kanban): listen to task.completed ... Refs: debug/BUG-...

## Dia 20 — brownfield
$ /sdd-feature comentários nas tarefas com menção @usuário

Fase 0 Discovery: db-reader mapeia tasks/users reais; code-improver
raio-X do módulo tasks; contrato as-is de notificações. Você valida.
Fase 1: adendo ao PRD (F06, wave nova, regressões a proteger).
Fases 2-6: artefatos com seção Compatibilidade → guarda de regressão
→ implementação → /commit por estágio → auditoria com db-reader
provando os CAs via SELECT → manifest ✔.

---

# End-to-end usage example (summarized)

*[Leia isto em Português](#exemplo-de-uso-ponta-a-ponta-resumido)*

## Day 1 — greenfield
$ /sdd-new kanban-based project management app for small teams

Phase 1: interview (short rounds) → PRD with F01-auth, F02-projects,
F03-kanban, F04-tasks, F05-notifications; Integration Matrix;
Waves: 0=[F01], 1=[F02], 2=[F03,F04], 3=[F05]. You approve → manifest.
Phase 2-3: Wave 0 artifacts + static audit APPROVED.
Phase 4: feature-implementer implements F01 stage by stage.
→ report: "Batches ready to commit: stage 1 (migrations),
stage 2 (routes), stage 3 (middleware+verification)".

$ /commit
Plan written to docs/commits/COMMITS-20261003-1430.md: 3 batches, each
with files, rationale and a ready message, for example:
        batch 1: db/migrations/001_users.sql, src/models/user.ts
        feat(auth): add user table and model
        ...
        SDD-Feature: F01 / SDD-Stage: 1
GATE: you review the file (you can edit batches and messages) and say ok
→ 3 commits in sequence, no interruptions → file updated with the hashes.

Phases 5-6: code-improver (2 findings, you approve 1) → integration
audit APPROVED → manifest ✔ → /commit docs(sdd) batch → Wave 1.

## Day 12 — bug with screenshot
$ /debug card disappears from kanban when task is completed [screenshot attached]

Phase 0: screenshot transcription ("PATCH /api/tasks/9f2 → 200; card
missing from Done column"); user's theory vs contract F04: the contract
says task.completed is emitted — spec F03 says the board listens for
board.updated.
Phase 4: classification = INTEGRATION (Matrix gap: wrong event).
Phase 5: sdd-feature-author fixes contracts F03/F04 + new scenario →
feature-implementer applies it → regression suite green.
$ /commit  → fix(kanban): listen to task.completed ... Refs: debug/BUG-...

## Day 20 — brownfield
$ /sdd-feature task comments with @user mentions

Phase 0 Discovery: db-reader maps real tasks/users; code-improver
X-rays the tasks module; as-is contract for notifications. You validate.
Phase 1: PRD addendum (F06, new wave, regressions to protect).
Phases 2-6: artifacts with a Compatibility section → regression guard
→ implementation → /commit per stage → audit with db-reader
proving ACs via SELECT → manifest ✔.
