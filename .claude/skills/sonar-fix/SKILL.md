---
name: sonar-fix
description: Lê os achados da extensão SonarQube for IDE (antigo SonarLint) no VS Code e aplica correções no código em lotes por regra, dividindo o trabalho entre dois modelos — o agente sonar-triager (Sonnet) lê os achados e prepara os lotes, e o agente sonar-fixer (Haiku) aplica e verifica cada lote. Use esta skill sempre que o usuário mencionar SonarQube, SonarLint, SonarQube for IDE, sonar, code smells, issues ou hotspots do sonar, "corrigir o que o sonar apontou", ou colar achados/logs da extensão.
---

# Sonar Fix — Correção guiada pela extensão SonarQube for IDE

Você é o **orquestrador**. O objetivo não é "silenciar o Sonar": é corrigir a causa que cada regra descreve, preservando o comportamento observável. A fonte dos achados é a extensão **SonarQube for IDE** no VS Code (não use servidor MCP do Sonar nem a Web API).

O trabalho é dividido entre dois agentes com modelos diferentes:

| Papel | Agente | Modelo | Faz |
|-------|--------|--------|-----|
| Leitura e preparo | `sonar-triager` | Sonnet | coleta os diagnósticos, estuda as regras, faz a triagem e grava `sonar/LOTES-*.md` |
| Programação | `sonar-fixer` | Haiku | aplica **um** lote por vez, verifica e marca as ocorrências |
| Orquestração | você (sessão principal) | o da sessão | fala com o usuário, aprova os lotes, escala e commita |

O plano `sonar/LOTES-<data>-<branch>.md` é a interface entre os agentes. Como o Haiku executa o plano sem reinterpretar as regras, a qualidade do plano define a qualidade das correções.

## Fase 0 — Preparar o contexto

1. Levante com o usuário (ou pelo pedido do comando): o escopo (padrão: arquivos alterados na branch atual, via `git diff --name-only` contra a branch base), a branch e se a extensão está em **connected mode** ou com as regras padrão.
2. A extensão só publica achados dos arquivos que analisou (em geral os abertos ou alterados). Peça ao usuário para abrir os arquivos-alvo ou rodar o comando da extensão que analisa os arquivos alterados antes da triagem.
3. Se o usuário colou itens do painel Problems, salve-os em `.sonar-fix/achados.txt` para o triager usar. Se ele mandou só o log do canal Output "SonarQube for IDE", use-o para destravar a análise (veja `references/vscode-extension.md`); o log não lista os achados.

## Fase 1 — Triagem (sonar-triager, Sonnet)

1. Invoque o agente `sonar-triager` com o escopo, a branch, o modo da extensão e o caminho de `.sonar-fix/achados.txt`, se existir.
2. Se ele retornar `SEM_ACHADOS`: a ferramenta de diagnósticos da IDE não está disponível para o agente ou os arquivos não foram analisados. Tente `getDiagnostics` na sessão principal (via ToolSearch); se ela funcionar aqui, salve os achados do Sonar em `.sonar-fix/achados.txt`. Se não funcionar, peça ao usuário para copiar o painel Problems filtrado por `sonar` (Ctrl+A, Ctrl+C). Depois, invoque o triager de novo.
3. Com `PLANO_PRONTO`, mostre ao usuário a tabela de lotes (lote | regra | ocorrências | arquivos | risco | requer-sonnet), os falsos positivos e os hotspots. **Aguarde a aprovação** e registre quais lotes foram aprovados. O usuário pode cortar, reordenar ou pedir para replanejar um lote (nesse caso, o triager é invocado de novo só para esse lote).

## Fase 2 — Correção (sonar-fixer, Haiku)

1. Antes do primeiro lote, rode a linha de base com os comandos de verificação do cabeçalho do plano e anote o que já falhava. Passe essa informação ao fixer, para ele não confundir falhas antigas com regressões.
2. Para cada lote aprovado, em ordem e **um de cada vez**, invoque `sonar-fixer` com o caminho do plano e o número do lote.
   - Lotes marcados com `requer-sonnet: sim` ou com risco alto: invoque o `sonar-fixer` com o modelo trocado para `sonnet` (parâmetro `model` do Agent) desde o início.
3. Trate o retorno:
   - `OK`: revise rapidamente o diff do lote (`git diff`) para confirmar que ficou restrito às ocorrências. Se estiver certo, ofereça o commit pelo fluxo `/commit` com o tipo sugerido. Depois, siga para o próximo lote.
   - `ESCALAR`: invoque de novo o `sonar-fixer` para o mesmo lote com o modelo `sonnet`. Se o motivo for um plano ambíguo, peça antes ao triager para replanejar o lote.
   - `FALHOU`: confirme que o fixer reverteu as edições. Então tente o lote uma vez com `sonnet`; se falhar de novo, pare e apresente o erro ao usuário.
   - Ocorrências `divergentes`: o código mudou desde a triagem. Junte-as e peça ao triager para replaná-las ao final.
   - Diagnósticos indisponíveis ou desatualizados: consulte `getDiagnostics` na sessão principal ou peça ao usuário para salvar ou reabrir os arquivos e mandar um novo print ou uma nova cópia do painel Problems.
4. Nunca rode dois fixers em paralelo: os lotes podem tocar os mesmos arquivos, e a verificação precisa isolar cada lote.

## Fase 3 — Security hotspots

Apresente ao usuário a seção de hotspots do plano. Só gere um lote de correção para um hotspot (via triager) se o usuário concordar; esses lotes são sempre de risco alto e vão direto para o `sonar-fixer` com `sonnet`. A mudança de status do hotspot na extensão ou no servidor é feita pelo usuário, nunca por você.

## Fase 4 — Relatório

1. Salve `sonar/SONAR-<data>-<branch>.md` com: escopo e modo da extensão, lotes aplicados (com o modelo que efetivamente corrigiu cada um: Haiku, ou Sonnet após escalada), achados corrigidos, falsos positivos com justificativa (para o usuário tratar na extensão ou no servidor), itens fora de escopo, divergentes e pendências. Referencie o plano `sonar/LOTES-*.md`.
2. Lembre ao usuário que a extensão cobre só o que analisou: o quality gate do servidor/CI é quem confirma o resultado final.
3. As próximas sessões começam pelos relatórios em `sonar/`: o triager os lê para não triar de novo falsos positivos já justificados.

## Regras da sessão
- Corrija a regra, não a métrica: nada de `// NOSONAR`, regras desativadas em `sonarlint.rules`, exclusões de arquivos ou mudanças no connected mode sem aprovação explícita.
- Nenhum código é editado antes de o usuário aprovar o plano de lotes.
- Um lote por vez; um lote só está concluído quando os testes, o lint e o build passam e o achado sumiu da extensão.
- Mudança de comportamento ou de API pública = pare e pergunte ao usuário.
- Os agentes nunca commitam; o commit é feito pela sessão principal via `/commit`, com aprovação do usuário.
