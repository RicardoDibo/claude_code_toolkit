# Template — plan.md

```markdown
# Plan — [F0X] [Nome da Feature]

## Contexto para o Implementador
- Leia antes de codar: contract.md (fronteiras), spec.md (como),
  discovery.md se existir (realidade do sistema)
- Contratos upstream: features/F0Y-*/contract.md
- NÃO altere saídas prometidas em contract.md sem aprovação humana

## Estágio 0 — Guarda de regressão (brownfield; omitir em greenfield)
- Tarefas: testes capturando o comportamento ATUAL dos fluxos listados
  em Compatibilidade (contract.md), antes de qualquer mudança
- Checkpoint: suite de regressão verde no código INTACTO

## Estágio 1 — [ex: Modelo de dados e migrations]
- Objetivo: [uma frase]
- Tarefas:
  - [ ] Criar migration X (db/migrations/...)
  - [ ] Criar model Y (src/models/...)
- Checkpoint: `npm run migrate && npm test -- models/task`
  (migration reversível com rollback limpo)
- Critérios do contrato cobertos: nenhum ainda (fundação)

## Estágio 2 — [ex: Rotas CRUD]
- Checkpoint: `curl -X POST .../api/tasks` → 201; sem token → 401
- Critérios cobertos: CA-1, CA-2

## Estágio N — Verificação de Contrato (obrigatório, sempre o último)
- [ ] Executar cada cenário de integração do contract.md
- [ ] Conferir tabela de Rastreabilidade do spec.md
- [ ] Marcar todos os CAs ✔ com evidência (log do teste/comando)
- [ ] Brownfield: guarda de regressão AINDA verde
- Checkpoint: tudo verde; atualizar manifest.md (Implementada ✔;
  "Contratos verificados" só após auditoria da wave)
```

Regras:
- 3 a 7 estágios; mais que isso = feature grande demais, proponha
  quebrá-la no PRD.
- Um estágio = uma sessão de trabalho verificável. Checkpoint sem
  comando executável não é checkpoint.
- Cada estágio concluído é um lote natural de commit (fluxo /commit).
