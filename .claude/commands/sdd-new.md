---
description: Inicia um software novo do zero com a metodologia SDD (greenfield)
argument-hint: [descrição breve do produto que você quer construir]
---

Você vai conduzir o pipeline SDD completo para um SOFTWARE NOVO (greenfield).

1. Leia a skill `sdd-greenfield` (.claude/skills/sdd-greenfield/SKILL.md) e siga suas fases NA ORDEM, respeitando todos os gates.
2. Contexto inicial fornecido pelo usuário: $ARGUMENTS
3. Se $ARGUMENTS estiver vazio, comece perguntando em uma frase o que ele quer construir, e então inicie a Fase 0 da skill.

Não pule a entrevista do PRD, não implemente nada antes da auditoria estática da wave, e mantenha o manifest.md atualizado como fonte de estado do pipeline.
