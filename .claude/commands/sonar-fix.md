---
description: Conecta ao servidor SonarQube (Community, Server ou Cloud), puxa os achados do projeto e aplica as correções em lotes verificados (triagem com Sonnet, correção com Haiku)
argument-hint: [conexão e/ou foco, ex.: "http://localhost:9000 projeto=meu-app", "só bugs", "novo código", "src/api"]
---

Você vai orquestrar a correção do código a partir dos achados do servidor SonarQube, lidos pela Web API.

1. Leia a skill `sonar-fix` (.claude/skills/sonar-fix/SKILL.md) e siga as fases NA ORDEM, respeitando os gates. Leia `references/webapi.md` antes da primeira chamada.
2. Pedido do usuário: $ARGUMENTS
3. Fase 0 é a conexão: pergunte URL (sugira http://localhost:9000), token e project key — sempre, sem gravar configuração no projeto — e sonde versão, autenticação, parâmetros válidos da versão (via /api/webservices/list), projeto, contagens e quality gate antes de mostrar o Gate 0.
4. A leitura e o preparo dos lotes são do agente `sonar-triager` (Sonnet). A aplicação de cada lote é do agente `sonar-fixer` (Haiku), com escalada para Sonnet quando o lote tiver risco alto, estiver marcado com `requer-sonnet` ou retornar `ESCALAR`/`FALHOU`.
5. Mostre o plano de lotes e aguarde aprovação antes de qualquer correção.

Lembretes invioláveis: nada avança sem o Gate 0 aprovado; o token nunca vai para o repositório nem para a linha de comando (use o config do curl no scratchpad e apague-o no fim); a skill NÃO escreve no servidor (transições, comentários e status de hotspot saem como comandos no relatório); corrija a regra, não a métrica (nada de NOSONAR, regras desativadas ou exclusões sem aprovação); um lote por vez, verificado LOCALMENTE com testes, lint e build (o servidor só é relido no rescan opcional da Fase 5); hotspots são revisados com o usuário, e você nunca os marca como seguros. Os agentes não commitam: lotes verdes são commitados pela sessão principal via /commit (fix | refactor | test), e a sessão termina com um relatório em docs/sonarqube/.
