# Template — spec.md

```markdown
# Spec — [F0X] [Nome da Feature]

## Visão Técnica
[2-3 parágrafos: abordagem, componentes principais, por quê]

## Modelo de Dados
### Tabela: tasks — ownership: ESTA feature — status: nova | existente (consome) | existente (altera — migração)
| Campo | Tipo | Constraints | Notas |
|-------|------|-------------|-------|
| id | UUID | PK | |
| project_id | UUID | FK → projects (F02, consumido) | |
| status | enum(todo, doing, done) | default: todo | máquina de estados abaixo |

Índices: (project_id, status)

### Máquina de Estados (se aplicável)
todo → doing → done; done → doing (reabrir, requer permissão X)

## API / Rotas
### POST /api/tasks
- Auth: Bearer token (middleware de F01)
- Request:  { "project_id": "uuid", "title": "string 1..200" }
- Response 201: { "id": "...", "status": "todo", ... }
- Erros: 400 (validação, campo errors[]), 401, 403 (não-membro), 404
[repetir por rota]

## Fluxos Internos
### Concluir tarefa (caso principal)
1. Valida token (F01) → 2. Verifica membership (F02) → 3. Transição →
4. Persiste → 5. Emite task.completed → 6. Responde 200

### Casos de borda (do PRD, Bloco C)
- Conclusão concorrente: [estratégia — ex: idempotente]
- Projeto deletado com tarefas órfãs: [estratégia]

## Validações e Segurança
| Rota | Authz | Validações |
|------|-------|------------|

## Dependências Técnicas
- Libs, serviços externos, env vars (nome + propósito, sem valores)

## Rastreabilidade
| Saída do contract.md | Onde este spec a produz |
|----------------------|--------------------------|
| evento task.completed | Fluxo "Concluir tarefa", passo 5 |
```

Rastreabilidade é a checagem anti-gap: promessa do contrato sem linha
aqui = spec incompleto.
