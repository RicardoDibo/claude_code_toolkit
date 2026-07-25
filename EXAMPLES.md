# Exemplo de uso ponta a ponta (resumido)

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
GATE 1: lote 1 staged (db/migrations/001_users.sql, src/models/user.ts)
        "fundação de dados da F01; restam 2 lotes". Você: ok
GATE 2: mensagem proposta:
        feat(auth): add user table and model
        ...
        SDD-Feature: F01 / SDD-Stage: 1
        Você: ok → commit. Loop para os lotes 2 e 3.

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
