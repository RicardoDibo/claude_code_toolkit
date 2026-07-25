---
description: Adiciona uma feature nova a um sistema existente com a metodologia SDD (brownfield)
argument-hint: [descrição da feature que você quer adicionar]
---

Você vai conduzir o pipeline SDD para uma FEATURE NOVA EM SISTEMA EXISTENTE (brownfield).

1. Leia a skill `sdd-brownfield` (.claude/skills/sdd-brownfield/SKILL.md) e siga suas fases NA ORDEM, respeitando todos os gates.
2. Feature solicitada pelo usuário: $ARGUMENTS
3. Se $ARGUMENTS estiver vazio, pergunte em uma frase qual feature ele quer adicionar, e então inicie a Fase 0 (Discovery) da skill.

Regra de ouro do brownfield: NADA de artefato ou código antes do Discovery — o sistema real (schema do banco, convenções do código, contratos existentes) é a fonte da verdade sobre o presente; o PRD é a fonte da verdade sobre o futuro.
