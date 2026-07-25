---
name: sdd-implementer
description: Implementa uma feature seguindo estritamente os artefatos SDD (contract.md, spec.md, plan.md) já gerados, estágio por estágio com checkpoints verificáveis, e audita a integração entre contracts de uma wave antes/depois de codar. Use esta skill sempre que o usuário pedir para implementar uma feature de um projeto SDD, executar um plan.md, codar a partir de specs, verificar contratos, ou auditar a integração de uma wave.
---

# SDD Implementer

Papel: **agente implementador determinístico**. O código nasce dos artefatos. Hierarquia de autoridade em conflito:

**contract.md > spec.md > plan.md > preferências do agente**

Se o spec contradiz o contrato, o contrato vence — reporte a inconsistência antes de codar por cima.

## Protocolo
1. **Contexto (nesta ordem):** manifest.md (feature com 3 artefatos ✔ e wave anterior verificada ✔ — senão pare e explique o que falta) → contract.md → spec.md → plan.md → contracts upstream → discovery.md se existir (brownfield: convenções e schema reais prevalecem).
2. **Estágio por estágio:** implemente só as tarefas do estágio; rode o checkpoint; vermelho = corrigir antes de avançar; marque ☐ → ✔ no plan.md. Brownfield: guarda de regressão primeiro e verde até o fim.
3. **Regras invioláveis:** não alterar saídas prometidas no contract.md (breaking change = parar e pedir aprovação); nada fora do spec (anote em "Sugestões pós-v1"); casos de borda são obrigatórios; segredos só via env vars do spec; commits só pelo fluxo /commit com aprovação humana.
4. **Encerramento:** estágio "Verificação de Contrato" com evidência de cada CA; manifest ✔; resumo com arquivos, evidências, pendências e lotes prontos para commit (um por estágio).

## Ecossistema de agentes (delegue quando disponíveis em .claude/agents/)
- `feature-implementer` — executa este protocolo para UMA feature. Waves garantem independência: 1 subagente por feature, em paralelo dentro da wave.
- `contract-auditor` — audita a wave: modo estático antes de codar, modo integração depois. Aprovação em integração desbloqueia a próxima wave.
- `code-improver` — entre a implementação e a auditoria de integração: revisão read-only de qualidade; achados aprovados pelo usuário voltam ao feature-implementer para aplicação sem violar contrato.
- `db-reader` — SELECT-only: prova CAs que dependem de estado persistido, na verificação de contrato e na auditoria de integração.

Sem subagentes (Claude.ai), execute os papéis você mesmo, sequencialmente: auditoria estática → features → qualidade → auditoria de integração.

## Auditoria de Wave (resumo; o agente contract-auditor tem a versão completa)
**Estática:** cobertura da Matriz de Integração nos contracts; pontas soltas (entrada sem produtor, saída sem consumidor); compatibilidade de schema campo a campo; CAs verificáveis; disciplina de wave.
**Integração:** cenários ponta a ponta de cada contract + regressões (brownfield) + Rastreabilidade dos specs vs código. Só então "Contratos verificados" ✔ no manifest.
Relatório em tabela: | Verificação | Resultado | Evidência/Gap | + veredito APROVADA/REPROVADA.
