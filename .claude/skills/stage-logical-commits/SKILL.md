---
name: stage-logical-commits
description: Analisa as mudanças não commitadas do repositório, agrupa em unidades lógicas coesas (vertical slices) e faz o stage de UM lote por vez, preparando commits profissionais. Use quando o usuário tiver várias mudanças acumuladas e pedir para organizar, separar, agrupar ou preparar commits — é o precursor natural da skill conventional-commits.
---

# Logical Commit Stager

Prepara commits pequenos e coesos a partir de um working directory bagunçado. Um lote staged = um conceito = um commit.

## Procedimento

1. **Levante o estado real** via Bash:
   - `git status --porcelain=v1` — modificados, untracked, deletados, renomeados
   - `git diff --stat` e, para entender conteúdo, `git diff` (por arquivo quando o diff total for grande)
   - `git diff --staged --stat` — se JÁ existe algo staged, pare e pergunte: incluir no próximo lote, commitar antes, ou desfazer o stage? Nunca misture silenciosamente.

2. **Exclusões (nunca entram em stage):**
   - `CLAUDE.md` (qualquer capitalização)
   - Diretórios `doc/`, `docs/`, `.claude/`, `.github/`, `.vscode/`
   - O próprio `.gitignore`
   - Qualquer coisa casando com padrões do `.gitignore` (cheque com `git check-ignore` em caso de dúvida)
   - Arquivos com cara de segredo (`.env*`, `*.pem`, `credentials*`, dumps): além de não stagear, ALERTE o usuário
   - Se o usuário pedir explicitamente para incluir algo desta lista, confirme uma vez e obedeça (exceto segredos — para esses, recomende template `.example`)

3. **Agrupe em vertical slices.** Cada lote é um conjunto pequeno de arquivos que juntos contam UMA mudança conceitual — ex: todos os arquivos do módulo "auth" através das camadas (Api, Application, Domain, Infra), não "todos os arquivos da pasta Domain". Critérios de corte, em ordem:
   - **Projeto SDD**: os estágios do `plan.md` da feature são a fronteira natural — um lote por estágio concluído (o checkpoint do estágio é a prova de que o lote é commitável). Consulte `features/F0X-*/plan.md` e o `manifest.md`.
   - Feature/módulo funcional (vertical slice)
   - Tipo de mudança (refactor puro separado de feature; formatação separada de lógica)
   - Dependência: se o lote B não compila sem o A, A vem primeiro
   - Arquivos de artefato SDD (`PRD.md`, `manifest.md`, `features/**`) formam lotes próprios (`docs(sdd): ...`), nunca misturados com código

4. **Regras de stage (invioláveis):**
   - NUNCA `git add .`, `git add -A`, `git add -u` nem diretórios inteiros grandes
   - Sempre arquivos individuais: `git add caminho/a caminho/b`
   - Um lote por invocação; mudanças parciais de um mesmo arquivo pertencendo a lotes diferentes: avise o usuário (hunk staging é interativo demais para automação — proponha o arquivo inteiro no lote dominante, ou deixe-o para stage manual)

5. **Apresente o lote staged**: lista de arquivos + justificativa do agrupamento em 1-2 frases + prévia dos lotes restantes (quantos e o tema de cada um).

6. **Handoff**: recomende invocar a skill `conventional-commits` para gerar a mensagem do lote atual. Após o commit, ofereça stagear o próximo lote.
