# Template — docs/commits/COMMITS-AAAAMMDD-HHMM.md

Plano de commits gerado pelo fluxo `/commit`. É a interface entre o agrupamento, a aprovação do usuário e a execução, e fica no repo como registro. Status válidos de um lote: `pendente`, `commitado — <hash>`, `falhou: <motivo>`.

````markdown
# Plano de commits — AAAA-MM-DD HH:MM

- Branch: <branch>
- Foco: <$ARGUMENTS ou "todas as mudanças">
- Estado: aguardando aprovação | em execução | concluído | interrompido no lote N

> Edite à vontade antes de aprovar: mova arquivos entre lotes, ajuste mensagens ou apague lotes inteiros.
> Ao dizer "ok" no chat, todos os lotes `pendente` são commitados em ordem, sem novas perguntas.

## Resumo

| # | Subject | Arquivos | Status |
|---|---------|----------|--------|
| 1 | feat(auth): add user table and model | 2 | pendente |
| 2 | feat(auth): add login and register routes | 3 | pendente |

## Lote 1 — <tema curto>

- Status: pendente
- Justificativa: <1-2 frases: por que esses arquivos formam uma mudança só>
- Depende de: — | Lote N
- Arquivos:
  - `db/migrations/001_users.sql` (novo)
  - `src/models/user.ts` (modificado)

```text
feat(auth): add user table and model

- Create users table with unique email index
- Add User model with password hash field

SDD-Feature: F01
SDD-Stage: 1
```

## Lote 2 — <tema curto>
...

## Fora do plano

- Excluídos pelas regras: `CLAUDE.md`, `.claude/settings.json`, ...
- Alertas: <segredos detectados, arquivos com mudanças parciais de mais de um lote (incluídos inteiros no lote dominante), etc.> | nenhum

## Log de execução

<!-- preenchido durante a execução, uma linha por evento -->
- Lote 1 → a1b2c3d
- Lote 2 → hook formatou src/routes/auth.ts; re-stage e novo commit → e4f5a6b

## Resultado

<commits criados (hash + subject); lotes não commitados e motivo; lembrete: commits locais, sem push>
````

## Regras do arquivo
- Um bloco ```text por lote, com a mensagem completa e literal que será passada ao `git commit`.
- Caminhos de arquivo em crases, um por linha, relativos à raiz do repo.
- A ordem dos lotes é a ordem de execução: se B depende de A, A vem antes.
- Durante a execução, atualize o `Status` do lote, a coluna da tabela-resumo e o log a cada commit, não só no final.
