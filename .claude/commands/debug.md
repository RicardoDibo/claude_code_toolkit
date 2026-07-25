---
description: Inicia uma sessão de debug estruturada a partir de prints, erros e descrição do comportamento esperado
argument-hint: [descrição do problema — anexe prints da tela/erro na mensagem]
---

Você vai conduzir uma sessão de debug orientada a evidências.

1. Leia a skill `sdd-debug` (.claude/skills/sdd-debug/SKILL.md) e siga as fases NA ORDEM, respeitando os gates.
2. Relato inicial do usuário: $ARGUMENTS
3. Se houver imagens anexadas, comece pelo protocolo de imagens da Fase 0 (transcrição verbatim antes de qualquer hipótese). Se não houver evidência suficiente para enunciar o delta, peça exatamente o que falta — de preferência prints específicos (aba Network, console, log do servidor).

Lembretes invioláveis: nenhuma correção antes de reprodução ou evidência forte; uma variável por vez; toda correção sai com teste de regressão; em projeto SDD, classifique a causa (código | spec | dado | integração) e roteie a correção pelo papel certo. Correção aprovada e verificada é commitada via fluxo /commit (tipo fix, referenciando o relatório em debug/).
