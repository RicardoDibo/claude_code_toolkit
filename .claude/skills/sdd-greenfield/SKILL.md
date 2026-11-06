---
name: sdd-greenfield
description: Orquestra o pipeline SDD completo para um software novo (greenfield), do PRD à implementação por waves, coordenando as skills prd-creator, sdd-feature-author e sdd-implementer e delegando aos agentes feature-implementer, contract-auditor, code-improver e db-reader. Use sempre que o usuário quiser começar um produto do zero com SDD, invocar /sdd-new, ou pedir para "criar um software novo com spec driven development".
---

# SDD Greenfield — Orquestrador

Você é o maestro do pipeline. Você NÃO faz o trabalho especialista — você sequencia fases, aciona a skill ou o agente certo em cada uma, verifica gates e mantém o estado no `manifest.md`. Cada fase termina com um gate explícito; nunca avance com gate aberto.

## Mapa de delegação

| Fase | Quem executa | Como |
|------|-------------|------|
| PRD | skill `prd-creator` | você mesmo, seguindo a skill |
| Artefatos por feature | skill `sdd-feature-author` | você mesmo, seguindo a skill |
| Auditoria (estática e integração) | agente `contract-auditor` | subagente |
| Implementação | agente `feature-implementer` | 1 subagente POR feature, em paralelo dentro da wave |
| Qualidade de código | agente `code-improver` | subagente, não edita código, grava `features/F0X-*/quality.md` |
| Provas de dados | agente `db-reader` | subagente, SELECT-only |
| Commits | fluxo `/commit` | thread principal, com aprovação do usuário |

## Fase 0 — Setup
Crie a estrutura base (se não existir): `PRD.md  manifest.md  features/`
Confirme com o usuário: nome do projeto, stack pretendida (se já souber), onde o código viverá.
**Gate:** estrutura criada.

## Fase 1 — PRD
Siga a skill `prd-creator`: entrevista iterativa (Blocos A-D) → `PRD.md` com Matriz de Integração e Execution Waves → `manifest.md`.
**Gate:** usuário aprova o PRD explicitamente. Registre a aprovação no manifest.

## Fase 2 — Artefatos da wave atual
Para cada feature da wave N (comece pela Wave 0), siga a skill `sdd-feature-author`: contract.md → spec.md → plan.md. Marque no manifest.
No greenfield o banco ainda não existe — o spec.md DEFINE o schema; não acione o db-reader aqui.
**Gate:** todas as features da wave com os 3 artefatos ✔.

## Fase 3 — Auditoria estática
Delegue ao agente `contract-auditor` em modo **estático** para a wave N. Se REPROVADA: volte à Fase 2 apenas para os artefatos apontados, e re-audite.
**Gate:** veredito APROVADA.

## Fase 4 — Implementação paralela
Para cada feature da wave, spawne um agente `feature-implementer` com: caminho do projeto, ID da feature, lista dos contracts upstream. Features da mesma wave são independentes por construção — é seguro rodar em paralelo. Colete os relatórios; qualquer conflito de artefato reportado por um implementador para a fase e volta ao usuário.
Ao receber os relatórios, ofereça ao usuário o fluxo `/commit` com os "lotes prontos para commit" reportados (um commit por estágio concluído).
**Gate:** todos os plan.md com checkpoints ✔ e estágio "Verificação de Contrato" verde.

## Fase 5 — Qualidade
Para cada feature implementada, delegue ao agente `code-improver` (alvo `feature F0X`, nível `high` salvo pedido do usuário). Ele grava `features/F0X-*/quality.md` com os achados ranqueados por severidade, cada um com cenário de falha e veredito. Apresente a tabela Resumo ao usuário, destacando os achados `crítica` e os que tocam contrato, e registre a decisão de cada achado (aplicar | dispensar) na coluna Decisão do arquivo. Para os achados aprovados, spawne novamente o `feature-implementer` da feature com a instrução de aplicar SOMENTE esses achados, sem violar o contract.md, e re-rodar os checkpoints afetados. Ofereça o fluxo `/commit` com os lotes reportados.
**Gate:** todo achado da rodada com Decisão preenchida e, nos aplicados, Resultado preenchido.

## Fase 6 — Auditoria de integração
Delegue ao `contract-auditor` em modo **integração** para a wave N. Para critérios de aceitação que se provam por dados persistidos, o auditor pode delegar SELECTs ao agente `db-reader` (agora o banco existe). Se REPROVADA: identifique se o gap é de artefato (→ Fase 2) ou de código (→ Fase 4 para a feature específica).
**Gate:** veredito APROVADA → marque "Contratos verificados" ✔ no manifest para a wave N.

## Fase 7 — Loop de waves
Sugira o lote final `docs(sdd)` via `/commit` (plan.md, manifest.md atualizados). Se existir Wave N+1 no PRD: volte à Fase 2 para ela. Senão: gere um resumo final (features entregues, CAs provados, achados dispensados, sugestões pós-v1 coletadas dos implementadores) e encerre.

## Regras do orquestrador
- O `manifest.md` é a máquina de estados do pipeline. Ao ser invocado num projeto que já tem manifest, LEIA-O primeiro e retome da fase correta em vez de recomeçar.
- Anuncie cada transição de fase em uma linha ("Fase 3 → auditoria estática da Wave 0").
- Interrupções do usuário no meio de uma fase não cancelam gates: registre a pendência e retome.
- Bug reportado durante o pipeline (print de erro, "não funciona") → fluxo `/debug` (skill sdd-debug); a correção volta roteada pelo papel certo.
- Mudança de escopo no meio do caminho (feature nova, requisito novo) = adendo ao PRD via `prd-creator` (entrevista curta) + reprocessar Matriz/Waves, nunca "encaixar direto" no código.
