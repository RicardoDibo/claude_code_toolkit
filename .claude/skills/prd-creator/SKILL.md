---
name: prd-creator
description: Conduz uma entrevista iterativa para conceber um novo produto ou funcionalidade complexa e gera um PRD completo no padrão Spec Driven Development (SDD), com features priorizadas, Execution Waves, matriz de integração e critérios de aceitação metrificáveis. Use esta skill sempre que o usuário quiser iniciar um novo produto, escrever um PRD, definir requisitos, planejar um roadmap de features, ou mencionar SDD, spec driven development, visão de produto ou documento de requisitos — mesmo que ele não use a palavra "PRD" explicitamente.
---

# PRD Creator (SDD)

Esta skill transforma o agente em um Product Engineer sênior que **extrai** a visão do produto do usuário através de entrevista iterativa e produz um PRD que serve de fonte da verdade para todo o pipeline SDD (PRD → Features → spec.md + plan.md + contract.md → Implementação).

**Princípio central: nunca adivinhe o produto.** O PRD nasce das respostas do usuário. Se uma informação essencial não foi dada, pergunte. Se o usuário não souber responder, proponha 2-3 opções concretas e peça para ele escolher — mas registre a decisão como "assumida, pendente de validação" no PRD.

**Modo enxuto (adendo)**: quando invocada por um orquestrador para UMA feature nova em PRD existente (brownfield), rode apenas os Blocos B e C focados na feature + integração + regressões a proteger, e produza um adendo (nova entrada em Features, novas linhas na Matriz de Integração, reposicionamento de Waves) em vez de um PRD inteiro.

## Fase 1 — Entrevista Iterativa

Rodadas curtas (2-4 perguntas por vez, nunca um questionário gigante). Se o usuário já colou um briefing, extraia o que puder e pergunte apenas os gaps. Blocos, nesta ordem:

### Bloco A — Visão e Problema
1. Que problema este produto resolve, e para quem? Como esse problema é resolvido hoje (workaround atual)?
2. Qual é o resultado observável de sucesso em 3-6 meses? (métrica, não feature)
3. O que este produto explicitamente NÃO fará na v1? (escopo negativo — evita scope creep)

### Bloco B — Público e Comportamento
4. Quem são as personas? Perfil comportamental de cada uma (frequência de uso, nível técnico, contexto — mobile/desktop, ambiente).
5. Qual é a jornada crítica? (o caminho que, se quebrar, o produto falha)

### Bloco C — Funcionamento (fecha os gaps de integração entre features)
6. Quais entidades de dados centrais existem e quem é dono de cada uma? (qual feature cria, qual consome?)
7. Que eventos importantes acontecem no sistema e quem precisa reagir a eles?
8. Quais regras de negócio cruzam mais de uma feature? (permissões, limites de plano, soft delete)
9. Estados e transições: existe máquina de estados central? (ex: pedido: rascunho → pago → enviado)
10. Casos de borda óbvios? (usuário deletado com dados órfãos, ação concorrente, offline)

### Bloco D — Restrições
11. Stack obrigatória ou preferida? Integrações externas obrigatórias?
12. Prazo, orçamento, compliance (LGPD etc.), performance?

**Critério de saída:** você consegue escrever cada user story sem inventar nada e desenhar o grafo de dependências entre features. Se não consegue, volte e pergunte.

## Fase 2 — Geração do PRD

Gere `PRD.md` com EXATAMENTE esta estrutura:

```markdown
# PRD — [Nome do Produto]
## 1. Resumo Executivo
## 2. Problema & Oportunidade
## 3. Objetivos e Métricas de Sucesso (cada objetivo com métrica-alvo)
## 4. Personas e Comportamento
## 5. Escopo Negativo (o que NÃO faremos na v1)
## 6. User Stories (Como [persona], eu quero [ação], para [valor] — cada
   story referencia a feature: → F01, F02...)
## 7. Modelo de Domínio Compartilhado
   - Entidades centrais, dono, consumidores
   - Eventos do sistema (nome, emissor, consumidores)
   - Regras de negócio transversais
## 8. Features — para cada (F01, F02...):
   - Nome, descrição, stories atendidas
   - Prioridade (P0/P1/P2) com justificativa
   - Dependências (← F0x) e dependentes (→ F0x)
   - Critérios de Aceitação metrificáveis ("login retorna token em
     < 500ms para credenciais válidas; 401 para inválidas" — nunca
     "deve funcionar bem")
## 9. Matriz de Integração — para cada PAR de features com dependência:
   | De | Para | O que atravessa a fronteira (dado/evento/chamada) | Onde será formalizado |
   ("Onde será formalizado" aponta para o contract.md da feature)
## 10. Execution Waves
   - Wave 0 (Foundations): features sem dependências
   - Wave 1, 2...: desbloqueadas pela wave anterior
   - Mesma wave = paralelismo seguro; justifique cada posicionamento
     pelo grafo de dependências
## 11. Restrições Técnicas
## 12. Decisões Assumidas (pendentes de validação)
## 13. Riscos e Perguntas em Aberto
```

**Regras de qualidade:**
- Toda feature precisa de ao menos um critério de aceitação verificável.
- Se duas features compartilham entidade ou evento e isso não está na Matriz de Integração, o PRD está incompleto — corrija antes de entregar.
- Wave 0 concentra infraestrutura de domínio (auth, modelos base, permissões). Feature que depende de auth nunca fica na Wave 0.

## Fase 3 — Harness de Qualidade

Após aprovação do PRD, gere o `manifest.md` na raiz:

```markdown
# Manifest — [Produto]
| Feature | Wave | spec.md | plan.md | contract.md | Implementada | Contratos verificados |
|---------|------|---------|---------|-------------|--------------|----------------------|
| F01 ... | 0    | ☐       | ☐       | ☐           | ☐            | ☐                    |
```

Nenhuma feature entra em implementação sem os 3 artefatos ✔; nenhuma wave N+1 começa sem "Contratos verificados" ✔ da wave N.

## Handoff

Próximo passo: skill **sdd-feature-author** para gerar os artefatos de cada feature, começando pela Wave 0. Estrutura:

```
projeto/
├── PRD.md
├── manifest.md
└── features/
    └── F01-autenticacao/
        ├── contract.md   ← escrito PRIMEIRO
        ├── spec.md
        └── plan.md
```
