---
name: conventional-commits
description: Gera uma mensagem de commit seguindo a especificação Conventional Commits a partir das mudanças atualmente em stage no repositório. Use quando o usuário pedir para escrever, gerar ou rascunhar uma mensagem de commit, mencionar conventional commits, ou após stagear um lote com a skill stage-logical-commits.
---

# Conventional Commits Generator

## Procedimento

1. **Leia o que está staged** via Bash: `git diff --staged --stat` e `git diff --staged`. Se não há nada staged, informe e sugira a skill `stage-logical-commits`. A mensagem descreve o QUE ESTÁ NO STAGE — nunca mudanças que você lembra ter feito mas não estão no diff.

2. **Determine o tipo:**
   | Tipo | Quando |
   |------|--------|
   | feat | nova funcionalidade |
   | fix | correção de bug |
   | docs | apenas documentação |
   | style | formatação sem mudança de significado |
   | refactor | nem fix nem feat |
   | perf | melhoria de performance |
   | test | testes adicionados/corrigidos |
   | build | build system ou dependências |
   | ci | configuração de CI |
   | chore | resto (não toca src nem test) |

   Se o diff mistura tipos (ex: feat + refactor não relacionado), sinalize — provavelmente o lote deveria ter sido dividido; ofereça voltar ao stager.

3. **Escopo** (opcional): derive dos módulos/diretórios tocados (`api`, `auth`, `ui`...). **Projeto SDD**: prefira o slug da feature (`feat(kanban): ...`) — siga o padrão já usado no histórico: cheque `git log --oneline -15`.

4. **Breaking change**: se o diff altera contrato público (rota renomeada, campo removido, assinatura mudada — em projeto SDD, qualquer saída prometida em `contract.md`), use `!` após tipo/escopo E rodapé `BREAKING CHANGE: descrição`. Em projeto SDD, breaking em saída de contrato exige aprovação prévia — se não houve, alerte antes de gerar a mensagem.

5. **Subject**: imperativo, minúsculas, sem ponto final, ≤ 72 caracteres, específico ("add retry with backoff to task sync", não "update code").

6. **Body** (lote substancial ou multi-arquivo): bullets do que mudou + o porquê quando não for óbvio. **Projeto SDD**: inclua estágio e evidência do checkpoint em uma linha (ex: "Plan F03 estágio 2 — checkpoint: testes de rotas verdes").

7. **Rodapés/trailers**: `Refs:` para issue/ticket se informado. Em SDD, opcionalmente `SDD-Feature: F03` e `SDD-Stage: 2` (rastreie com `git log --grep`). NUNCA inclua `Co-Authored-By: * <noreply@anthropic.com>` nem co-autoria não fornecida explicitamente pelo usuário.

8. **Saída**: mensagem completa em bloco ```text para cópia. Não execute `git commit` — quem decide é o usuário (ou o fluxo /commit, que pede aprovação explícita).

## Exemplo

```text
feat(kanban): add drag-and-drop column reordering

- Implement reorder endpoint PATCH /api/boards/:id/columns
- Persist column order as fractional index to avoid mass updates
- Emit board.updated event consumed by F05 dashboard

Plan F03 estágio 3 — checkpoint: cenário de integração com F05 verde
SDD-Feature: F03
SDD-Stage: 3
```
