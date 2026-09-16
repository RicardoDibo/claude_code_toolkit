---
description: Lê os achados da extensão SonarQube for IDE no VS Code e aplica as correções em lotes verificados (triagem com Sonnet, correção com Haiku)
argument-hint: [foco, ex.: "arquivos alterados", "só bugs", "src/api" — ou cole os itens do painel Problems]
---

Você vai orquestrar a correção do código a partir dos achados da extensão SonarQube for IDE (VS Code).

1. Leia a skill `sonar-fix` (.claude/skills/sonar-fix/SKILL.md) e siga as fases NA ORDEM, respeitando os gates.
2. Pedido do usuário: $ARGUMENTS
3. A leitura e o preparo dos lotes são do agente `sonar-triager` (Sonnet). A aplicação de cada lote é do agente `sonar-fixer` (Haiku), com escalada para Sonnet quando o lote tiver risco alto, estiver marcado com `requer-sonnet` ou retornar `ESCALAR`/`FALHOU`.
4. Mostre o plano de lotes e aguarde aprovação antes de qualquer correção.

Lembretes invioláveis: corrija a regra, não a métrica (nada de NOSONAR, regras desativadas ou exclusões sem aprovação); um lote por vez, verificado com testes, lint e build e com uma nova leitura dos diagnósticos; hotspots são revisados com o usuário, e você nunca os marca como seguros. Os agentes não commitam: lotes verdes são commitados pela sessão principal via /commit (fix | refactor | test), e a sessão termina com um relatório em sonar/.
