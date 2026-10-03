---
name: conventional-commits
description: Gera mensagens de commit seguindo a especificação Conventional Commits a partir das mudanças em stage ou, no plano do /commit, do diff de cada lote planejado. Use quando o usuário pedir para escrever, gerar ou rascunhar uma mensagem de commit, mencionar conventional commits, ou após stagear um lote com a skill stage-logical-commits.
---

# Conventional Commits Generator

## Procedimento

1. **Leia o diff do lote** via Bash:
   - **Modo avulso**: `git diff --staged --stat` e `git diff --staged`. Se não há nada staged, informe e sugira a skill `stage-logical-commits`.
   - **Modo plano** (fluxo `/commit`, nada staged ainda): para cada lote, `git diff --stat -- <arquivos do lote>` e `git diff -- <arquivos do lote>`; arquivos untracked não aparecem no diff, então leia-os do disco.

   A mensagem descreve O QUE ESTÁ NO DIFF DO LOTE — nunca mudanças que você lembra ter feito mas não estão nele.

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

8. **Saída**: mensagem completa em bloco ```text. No modo avulso, ela vai para o chat, para cópia. No modo plano, vai para a seção do lote no plano `docs/commits/COMMITS-*.md`, e a aprovação é o gate único do `/commit`. Não execute `git commit` — quem decide é o usuário (ou o fluxo /commit, depois que o plano é aprovado).

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
