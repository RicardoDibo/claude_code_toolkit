---
description: Revisa o diff, uma branch, uma feature SDD ou um caminho no molde do /code-review (correção primeiro, depois reúso, simplificação, eficiência, altitude e testes) e, com --fix, aplica os achados aprovados
argument-hint: [low|medium|high|xhigh] [alvo: vazio=diff | branch | F0X | caminho] [--fix] [--max-findings n|all]
---

Você vai orquestrar uma revisão de código.

1. Leia a skill `code-improve` (.claude/skills/code-improve/SKILL.md) e siga as fases NA ORDEM, respeitando os gates.
2. Pedido do usuário: $ARGUMENTS
3. A revisão é do agente `code-improver`, que grava o relatório em `features/F0X-*/quality.md` ou `quality/QUALITY-*.md`. A aplicação, só com `--fix`, é do agente `feature-implementer`.
4. Mostre a tabela de achados por severidade e, com `--fix`, aguarde a escolha dos achados antes de qualquer correção.

Lembretes invioláveis: todo achado relatado tem cenário de falha concreto e veredito (CONFIRMED ou PLAUSIBLE); nenhum código muda sem aprovação, e achado que toca uma saída prometida no contract.md exige aprovação da mudança de contrato; nada é publicado em PR nem enviado por push. Os agentes não commitam: as correções são commitadas pela sessão principal via /commit (fix | perf | test | refactor).
