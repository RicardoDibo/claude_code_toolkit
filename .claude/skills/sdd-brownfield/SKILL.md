---
name: sdd-brownfield
description: Orquestra o pipeline SDD para adicionar uma feature nova a um sistema existente (brownfield), começando por um Discovery obrigatório do banco e do código real antes de qualquer artefato, coordenando prd-creator, sdd-feature-author e sdd-implementer e delegando aos agentes db-reader, code-improver, contract-auditor e feature-implementer. Use sempre que o usuário quiser adicionar, estender ou modificar uma funcionalidade em um sistema que já existe, invocar /sdd-feature, ou mencionar "feature nova no sistema atual".
---

# SDD Brownfield — Orquestrador

Você é o maestro do pipeline para features em sistemas vivos. A diferença fundamental para o greenfield: **o presente já está decidido**. O schema real, as convenções do código e os contratos existentes são fatos, não escolhas — e o Discovery vem antes de tudo, porque spec escrito de memória é a principal fonte de retrabalho em brownfield.

## Mapa de delegação
Igual ao greenfield (skills prd-creator / sdd-feature-author / sdd-implementer; agentes feature-implementer, contract-auditor, code-improver, db-reader; commits via fluxo /commit no thread principal), com o db-reader promovido a protagonista da Fase 0.

## Fase 0 — Discovery (obrigatória, nada de artefato antes dela)
Monte o retrato do terreno que a feature vai tocar:

1. **Banco real** — delegue ao agente `db-reader`: tabelas relacionadas ao domínio da feature (INFORMATION_SCHEMA / SHOW), colunas com tipos e constraints, FKs, índices, volumes aproximados e amostras de distribuição quando relevante (ex: valores distintos de um campo status).
2. **Código real** — usando Glob/Grep, mapeie: módulos que a feature vai tocar ou consumir, convenções (naming, error handling, camadas), pontos de extensão existentes (middlewares, eventos, services).
3. **Convenções e dívidas da vizinhança** — delegue ao agente `code-improver` em modo `discovery`, com os módulos vizinhos identificados como alvo. O relatório (convenções observadas + dívidas com cenário de falha) serve como raio-X da região: consolide-o no discovery.md e não aplique nada agora; é insumo para o spec e o plan.
4. **Contratos existentes** — se o projeto já é SDD, leia `PRD.md`, `manifest.md` e os `contract.md` das features vizinhas. Se NÃO é SDD, gere um **contrato as-is** por área tocada: um `features/_asis/<area>/contract.md` de engenharia reversa (entradas/saídas/regras observadas no código e no banco), marcado como "as-is, não aprovado como intenção". Ele vira a referência de compatibilidade.

Produza um `features/F0X-<nome>/discovery.md` consolidando os 4 itens.
**Gate:** discovery.md revisado pelo usuário ("é isso mesmo que existe hoje?").

## Fase 1 — Adendo ao PRD
Siga a skill `prd-creator` em modo enxuto — entrevista curta focada na feature:
- O que a feature faz e para quem (stories);
- **Integração**: o que ela consome do existente, o que expõe, o que dispara (alimenta a Matriz de Integração com linhas novas, incluindo pares feature-nova ↔ áreas as-is);
- **Regressões a proteger**: comportamentos atuais que NÃO podem mudar (vira critérios de aceitação negativos);
- Posicionamento em wave (quase sempre uma wave nova dependente das áreas tocadas).
Se não existe PRD.md, crie um retroativo mínimo (visão em 1 parágrafo + features as-is listadas) antes do adendo.
**Gate:** usuário aprova o adendo.

## Fase 2 — Artefatos com compatibilidade
Siga a skill `sdd-feature-author` para a feature, com três exigências extras do brownfield:
- O `contract.md` ganha a seção **Compatibilidade**: contratos/áreas as-is consumidos e os critérios de regressão da Fase 1;
- O `spec.md` referencia o schema REAL do discovery.md: cada tabela marcada como `existente (consome)`, `existente (altera — requer migração)` ou `nova`. Alterações em tabelas existentes exigem plano de migração reversível e checagem de impacto (quem mais usa essa coluna? → Grep + db-reader);
- O `plan.md` inclui um estágio inicial "Guarda de regressão": testes que capturam o comportamento atual ANTES de mudar qualquer coisa.
**Gate:** os 3 artefatos ✔ no manifest.

## Fase 3 — Auditoria estática ampliada
Delegue ao `contract-auditor` (modo estático) com uma verificação extra: cada campo/tabela citado no spec.md existe de fato conforme o discovery.md? Divergência = voltar à Fase 2 (ou refazer discovery se o banco mudou).
**Gate:** APROVADA.

## Fase 4 — Implementação
Spawne o agente `feature-implementer` para a feature. Ordem inviolável: guarda de regressão verde primeiro, depois os demais estágios. Instrua-o a seguir as convenções documentadas no discovery.md (consistência com o vizinho > preferência própria). Ao receber o relatório, ofereça o fluxo `/commit` com os lotes prontos (um por estágio).
**Gate:** checkpoints ✔, incluindo a guarda de regressão ainda verde ao final.

## Fase 5 — Qualidade
Agente `code-improver` (alvo `feature F0X`) sobre o código novo E os arquivos existentes alterados; ele grava `features/F0X-*/quality.md`. Registre a decisão do usuário em cada achado; os aprovados voltam ao `feature-implementer` para aplicação sem violar contrato nem regressões (guarda de regressão verde ao final).
**Gate:** todo achado da rodada com Decisão preenchida e, nos aplicados, Resultado preenchido.

## Fase 6 — Auditoria de integração
`contract-auditor` em modo integração: cenários do contract.md + critérios de regressão, com `db-reader` provando estados persistidos (antes/depois via SELECT).
**Gate:** APROVADA → manifest ✔ ("Implementada" e "Contratos verificados").

## Fase 7 — Encerramento
Lote final `docs(sdd)` via `/commit`. Resumo: o que foi entregue, provas de CAs e de regressão, dívidas encontradas na vizinhança (relatório da Fase 0/5) que ficaram para depois, e atualização final do PRD/manifest.

## Regras do orquestrador
- Manifest como máquina de estados; retome de onde parou.
- Se durante QUALQUER fase o código/banco real contradisser o discovery.md, pare, atualize o discovery e reavalie os artefatos — nunca implemente por cima de um retrato desatualizado.
- Bug reportado no caminho → fluxo `/debug` (skill sdd-debug).
- Anuncie transições de fase em uma linha.
