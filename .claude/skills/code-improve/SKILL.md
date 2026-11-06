---
name: code-improve
description: Revisa código no mesmo molde do /code-review, delegando ao agente code-improver — bugs de correção primeiro, depois reúso, simplificação, eficiência, altitude e cobertura de testes, cada achado com cenário de falha, veredito e severidade — e, com --fix, aplica os achados aprovados pelo feature-implementer. Use sempre que o usuário invocar /code-improve, pedir revisão de código, code review, "revisa meu diff", "tem bug aqui?", "o que dá para melhorar neste código", ou quiser revisar uma branch, uma feature SDD ou uma pasta antes de commitar.
---

# Code Improve — Revisão de código fora (ou dentro) do pipeline SDD

Você é o **orquestrador**. A revisão é do agente `code-improver`; a aplicação dos achados é do agente `feature-implementer`. Você interpreta o pedido, fala com o usuário, segura os gates e encaminha os commits. O relatório em disco é a interface entre os dois agentes.

| Papel | Agente | Faz |
|-------|--------|-----|
| Revisão | `code-improver` | revisa o alvo, verifica cada achado e grava o relatório |
| Aplicação (só com `--fix`) | `feature-implementer` | aplica os achados aprovados, roda a verificação e preenche o Resultado |
| Orquestração | você (sessão principal) | argumentos, gates, apresentação e commits |

## Fase 0 — Interpretar os argumentos

Sintaxe, no mesmo formato do /code-review: `[nível] [alvo] [--fix] [--max-findings <n>|all]`.

- **Nível:** `low | medium | high | xhigh`, sempre o primeiro argumento quando presente. Sem nível, use `high`. `max` e `ultra` do /code-review não existem aqui: se pedidos, use `xhigh` e avise.
- **Alvo:**
  - vazio → `diff` (`git diff HEAD` + arquivos não rastreados);
  - número de PR ou nome de branch → `branch <base>` com a branch atual comparada à base informada (ou à base padrão do repositório). Se o alvo for uma branch que não está em checkout, peça ao usuário para fazer o checkout; você não troca de branch;
  - `F0X` ou nome de feature presente no `manifest.md` → `feature F0X`;
  - caminho de arquivo ou pasta → `caminhos`.
- **`--max-findings <n>`** sobrescreve o máximo do nível; `all` remove o corte.
- **`--fix`** pede a aplicação dos achados depois da revisão (Fase 3). Sem ele, a sessão termina no relatório.
- `--comment` e `--post` não existem aqui: o kit não publica nada em PR. Diga isso ao usuário e siga só com o relatório.

Se o alvo `diff` estiver vazio, diga que não há mudanças para revisar e pare.

## Fase 1 — Revisão (code-improver)

1. Invoque o agente `code-improver` com o modo `revisão`, o alvo, o nível e o máximo de achados.
2. Trate o retorno:
   - `SEM_ALVO`: confirme o alvo com o usuário e invoque de novo.
   - `SEM_ACHADOS`: informe o alvo revisado, o nível e quantos candidatos foram descartados. Encerre.
   - `RELATORIO_PRONTO`: siga para a Fase 2.

## Fase 2 — Apresentação

Mostre ao usuário a tabela Resumo do relatório, ordenada por severidade, com o caminho do arquivo (`features/F0X-*/quality.md` ou `quality/QUALITY-*.md`). Destaque à parte:
- os achados `crítica`;
- os achados com `toca contrato: sim`, explicando que corrigi-los muda uma saída prometida no contract.md;
- quantos achados ficaram de fora pelo máximo, para o usuário decidir se quer um nível maior.

Sem `--fix`: encerre aqui. Diga que os achados podem ser aplicados depois pedindo para aplicar IDs do relatório (ex.: "aplica Q1 e Q2 do quality/QUALITY-...md"): nesse caso, a sessão retoma direto deste gate com o relatório existente, sem nova revisão, desde que os arquivos do alvo não tenham mudado desde então (confira com `git diff`; se mudaram, faça uma nova rodada).

**Gate (só com `--fix`):** pergunte quais achados aplicar. Uma única resposta basta ("todos", "todos menos Q3", "só Q1 e Q2"). Achados que tocam contrato só entram com aprovação explícita da mudança de contrato; em projeto SDD, essa mudança volta para a skill `sdd-feature-author` antes de qualquer código. Registre a decisão de cada achado na coluna Decisão do relatório (aplicar | dispensar). Nenhum código é alterado antes desta resposta.

## Fase 3 — Aplicação (feature-implementer, só com `--fix`)

1. Rode a linha de base com os comandos de verificação do cabeçalho do relatório e anote o que já falhava.
2. Invoque o `feature-implementer` no modo "aplicar achados", com o caminho do relatório e a linha de base. Fora de uma feature SDD, informe que não há contract.md: a autoridade é o comportamento observável atual, coberto pelos testes.
3. Com o retorno, confira o diff (`git diff`) para garantir que ficou restrito aos achados aprovados e que o campo Resultado de cada um foi preenchido. Achado `não aplicado` volta ao usuário com o motivo.
4. Ofereça o fluxo `/commit` com os lotes reportados (um por tipo: `fix`, `perf`, `test`, `refactor`).

## Fase 4 — Encerramento

Resuma: alvo, nível, achados relatados, aplicados, dispensados e não aplicados, e o caminho do relatório, que fica como registro. Em projeto SDD, se o alvo foi uma feature, lembre que a auditoria de integração do `contract-auditor` ainda é o que libera a wave.

## Regras da sessão
- O code-improver nunca edita código; só o feature-implementer edita, e só com `--fix` depois do gate.
- Nenhuma correção sem aprovação do usuário, nem mesmo de achado `crítica`.
- Nada é publicado fora do repositório: sem comentário em PR, sem push.
- Os agentes nunca commitam; o commit é feito pela sessão principal via `/commit`, com aprovação do usuário.
- Um achado sem cenário de falha não é apresentado como defeito. Se o usuário discordar de um achado, registre `dispensar` com o motivo no relatório para a próxima rodada não repeti-lo.
