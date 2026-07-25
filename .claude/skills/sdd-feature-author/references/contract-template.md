# Template — contract.md

```markdown
# Contract — [F0X] [Nome da Feature]

## Identificação
- ID: F0X | Wave: N
- Depende de: ← F0Y (o quê exatamente)
- Desbloqueia: → F0Z (o quê exatamente)

## Entradas
| Nome | Tipo (dado/evento/chamada) | Origem | Formato/Schema | Obrigatório |
|------|---------------------------|--------|----------------|-------------|
| user_id | dado | F01-auth (saída: sessão) | UUID | sim |

## Saídas (promessas públicas — mudanças aqui são breaking changes)
| Nome | Tipo | Consumidores | Formato/Schema | Estabilidade |
|------|------|--------------|----------------|--------------|
| evento task.completed | evento | F04, F05 | { task_id, user_id, completed_at } | estável |

## Regras e Invariantes
- [Regra que a feature garante SEMPRE, ex: "uma tarefa só pode ser
  concluída por membro do projeto"]

## Critérios de Aceitação (Dado / Quando / Então)
- [ ] Dado usuário autenticado membro do projeto, quando conclui tarefa,
      então status = done e evento task.completed é emitido em < 1s.
- [ ] Dado usuário sem permissão, quando tenta concluir, então 403 e
      nenhum evento é emitido.

## Compatibilidade (brownfield — omitir em greenfield)
- Áreas/contratos as-is consumidos: [features/_asis/...]
- Critérios de regressão:
  - [ ] Dado o fluxo existente X, quando esta feature está ativa,
        então X continua retornando Y (inalterado).

## Cenários de Integração (um por vizinho na Matriz de Integração)
### Com F01 (auth)
1. Login → token → chamada a esta feature com token → 200
2. Token expirado → 401 (esta feature NÃO reimplementa validação; usa
   middleware exposto por F01)

## Fora do Contrato (explicitamente não prometido)
- [Coisas que consumidores NÃO devem assumir]
```

Dicas:
- Schemas com exemplos concretos valem mais que prosa.
- Saída sem consumidor conhecido: questione se deveria existir (YAGNI) ou se falta linha na Matriz de Integração do PRD.
- "Fora do Contrato" evita acoplamento acidental.
