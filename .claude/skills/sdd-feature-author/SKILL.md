---
name: sdd-feature-author
description: Gera os três artefatos SDD de uma feature — contract.md (contrato de integração), spec.md (especificação técnica) e plan.md (plano de ação por estágios) — a partir de um PRD existente. Use esta skill sempre que o usuário pedir para especificar, detalhar ou preparar uma feature para implementação, criar spec/plan/contract, ou avançar uma wave do pipeline SDD depois que o PRD estiver aprovado.
---

# SDD Feature Author

Gera os artefatos de uma feature no pipeline SDD. A ordem de escrita é **sempre esta**:

**1º contract.md → 2º spec.md → 3º plan.md**

O contrato vem primeiro porque define as fronteiras (o que entra, o que sai, o que a feature promete às outras). O spec detalha COMO cumprir o contrato. O plan sequencia a implementação. Spec antes do contrato é a receita para features que "existem isoladas mas não se integram".

## Pré-requisitos
1. Leia o `PRD.md` — em especial Modelo de Domínio, Matriz de Integração e a entrada da feature. Sem PRD, pare e sugira a skill `prd-creator`.
2. Leia os `contract.md` de todas as features das quais esta depende (← no PRD). As saídas delas são as entradas desta.
3. Se existir `features/F0X-*/discovery.md` (brownfield), ele é a fonte da verdade sobre o schema e as convenções REAIS — o spec referencia o que existe, não o que você imagina.
4. Dado faltante que não está no PRD nem nos contratos upstream: pergunte ao usuário; não invente.

## Artefato 1 — contract.md
Template completo em `references/contract-template.md` (leia antes de gerar). Essencial: identificação (ID, wave, ←/→), Entradas (origem, formato, obrigatoriedade), Saídas (consumidores, schema estável — promessa pública), Regras/invariantes, Critérios de aceitação (Dado/Quando/Então), Cenários de integração (≥1 por vizinho na Matriz), e "Fora do Contrato".
**Brownfield:** adicione a seção **Compatibilidade** — contratos/áreas as-is consumidos e critérios de regressão (comportamentos existentes que não podem mudar).

**Checagem de gap obrigatória:** cruze o contrato com a Matriz de Integração. Toda linha da matriz envolvendo esta feature aparece aqui como entrada ou saída. Toda entrada tem produtor (contrato upstream, área as-is ou input externo declarado). Ponta solta = reporte antes de prosseguir.

## Artefato 2 — spec.md
Template em `references/spec-template.md`. Essencial: visão técnica, modelo de dados (com ownership), API/rotas com exemplos e erros, fluxos internos incluindo casos de borda do PRD, validações/segurança por rota, dependências técnicas, e a tabela de **Rastreabilidade** (toda saída do contract → mecanismo no spec que a produz).
**Brownfield:** cada tabela marcada como `existente (consome)`, `existente (altera — requer migração reversível + checagem de impacto)` ou `nova`, conforme o discovery.md.

## Artefato 3 — plan.md
Template em `references/plan-template.md`. Estágios pequenos (3-7) e verificáveis:
- Cada estágio termina com checkpoint executável (teste, curl, script). Sem checkpoint = não é estágio.
- **Brownfield:** o primeiro estágio é sempre "Guarda de regressão" (testes que capturam o comportamento atual ANTES de mudar qualquer coisa).
- O último estágio é sempre "Verificação de Contrato": rodar os cenários de integração do contract.md.
- Estágio que depende de artefato de outra feature não implementada = violação de wave; sinalize.

## Ao concluir
1. Atualize o `manifest.md` (artefatos ✔).
2. Próximo passo: implementação via agente `feature-implementer` (ou skill `sdd-implementer`).
3. Se todas as features da wave têm os 3 artefatos, sugira rodar o agente `contract-auditor` em modo estático antes de codar.
