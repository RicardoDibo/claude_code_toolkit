---
name: sonar-fix
description: Conecta ao servidor SonarQube (Community Build, Server ou Cloud) pela Web API, puxa os achados do projeto e aplica correções no código em lotes por regra, dividindo o trabalho entre dois modelos — o agente sonar-triager (Sonnet) lê o servidor, estuda as regras e prepara os lotes, e o agente sonar-fixer (Haiku) aplica e verifica cada lote. Use esta skill sempre que o usuário mencionar SonarQube, SonarLint, SonarQube for IDE, sonar, code smells, issues, hotspots ou quality gate do sonar, "corrigir o que o sonar apontou", der uma URL de servidor Sonar (ex.: http://localhost:9000), ou colar achados do painel Problems.
---

# Sonar Fix — Correção guiada pelo servidor SonarQube

Você é o **orquestrador**. O objetivo não é "silenciar o Sonar": é corrigir a causa que cada regra descreve, preservando o comportamento observável.

**Fonte canônica dos achados: a Web API do servidor SonarQube.** A extensão SonarQube for IDE no VS Code é complemento — serve como verificação local rápida entre lotes, quando estiver disponível. Antes de qualquer coisa, leia `.claude/skills/sonar-fix/references/webapi.md`; leia `references/vscode-extension.md` só se for usar os diagnósticos da IDE.

O trabalho é dividido entre dois agentes com modelos diferentes:

| Papel | Agente | Modelo | Faz |
|-------|--------|--------|-----|
| Leitura e preparo | `sonar-triager` | Sonnet | puxa as issues do servidor, estuda as regras, faz a triagem e grava `docs/sonarqube/LOTES-*.md` |
| Programação | `sonar-fixer` | Haiku | aplica **um** lote por vez, verifica localmente e marca as ocorrências |
| Orquestração | você (sessão principal) | o da sessão | conecta, fala com o usuário, aprova os lotes, escala, reanalisa e commita |

O plano `docs/sonarqube/LOTES-<data>-<projeto>.md` é a interface entre os agentes. Como o Haiku executa o plano sem reinterpretar as regras, a qualidade do plano define a qualidade das correções.

## Fase 0 — Conexão (Gate 0)

A conexão é perguntada **a cada sessão**. Não leia nem grave configuração de conexão no projeto.

1. Pergunte ao usuário, aceitando o que ele já tenha passado em `$ARGUMENTS`:
   - **URL** do servidor (sugira `http://localhost:9000`, o caso mais comum de container local);
   - **token** (User token; explique que ele não será gravado no repositório);
   - **project key** (se ele não souber, descubra com `/api/components/search?qualifiers=TRK` depois de autenticar).
2. Grave o token num arquivo de config do curl **no scratchpad da sessão** (nunca no projeto) e sonde, nesta ordem, parando no primeiro erro para tratá-lo com o usuário:
   - `/api/system/status` (alcance) e `/api/server/version` (versão);
   - autenticação: `/api/authentication/validate` com Bearer e, em `401`, basic auth — registre o que funcionou e **o prefixo do token** (`squ_` user, `sqa_`/`sqp_` análise: com token de análise, `project_analyses/search` e o write-back dão 403);
   - `/api/webservices/list`: anote os nomes de parâmetro válidos nesta versão (`componentKeys` vs `components`, `types`/`severities` vs `impactSeverities`, `statuses` vs `issueStatuses`, `sinceLeakPeriod` vs `inNewCodePeriod`) e se as ações de `hotspots` ainda existem;
   - `/api/components/search` (projeto existe), `/api/measures/component` (contagens), `/api/qualitygates/project_status` (gate);
   - `/api/project_branches/list?project=<key>`: **liste** as branches e pegue a principal e o `analysisDate`. Nunca descubra suporte a branch mandando `&branch=` e olhando o código HTTP — o parâmetro é aceito e filtra, e uma branch inexistente devolve `total=0` **sem erro**, que parece projeto limpo. Só use um nome que esteja na lista, ou omita.
3. Baixe os achados do escopo e **verifique o frescor por conteúdo, não por data** (veja "Frescor" em `references/webapi.md`): corte cada `textRange` no arquivo local e conte quantas ocorrências têm trecho coerente, quantas estão deslocadas e quantas apontam para arquivo/linha que não existe. Data de análise mais velha que o último commit **não** significa dado velho: um scan de working tree sujo ou sem `sonar.branch.name` fica com data e rótulo de branch enganosos e ainda assim descreve o código do disco.
4. Escreva o **perfil da sessão** num arquivo do scratchpad (URL, project key, versão, esquema de auth e tipo do token, nomes de parâmetro válidos, branches existentes, resultado da verificação de frescor, caminho do config do curl). É o que você passa ao triager, para ele não repetir a sondagem.

**Gate 0:** mostre ao usuário versão do servidor, projeto, contagens por tipo (bugs, vulnerabilidades, code smells, hotspots), status do quality gate, data e branch da última análise, e **quantas ocorrências do escopo foram localizadas no código local**. Se a taxa de deslocamento for alta, ofereça o rescan já aqui, antes de triar. Nada avança sem aprovação.

## Fase 1 — Escopo

Ofereça as opções e registre quantas issues ficaram no escopo:

- **new code period** (`inNewCodePeriod` / `sinceLeakPeriod`) — o padrão quando o projeto é grande;
- **arquivos alterados na branch**: `git diff --name-only` contra a branch base, cruzado com os `component` das issues;
- **caminho ou módulo** (`componentKeys=<key>:src/api`);
- **regra ou severidade específica** (`rules=`, `impactSeverities=`/`severities=`);
- **tudo** — avise do teto de ~10.000 issues da API e estreite se `total` chegar perto.

## Fase 2 — Triagem (sonar-triager, Sonnet)

1. Invoque o `sonar-triager` com: caminho do perfil da sessão, escopo, branch e os relatórios anteriores em `docs/sonarqube/`.
2. Se ele retornar `SEM_ACHADOS`, o motivo agora é escopo vazio, filtro inválido na versão ou projeto sem análise — **não** é "arquivo não analisado". Cruze a contagem com `/api/measures/component`, ajuste o escopo ou o filtro com o usuário e reinvoque.
3. Com `PLANO_PRONTO`, mostre a tabela de lotes (lote | regra | ocorrências | arquivos | risco | requer-sonnet), os falsos positivos e os hotspots. **Aguarde a aprovação** e registre quais lotes foram aprovados. O usuário pode cortar, reordenar ou pedir para replanejar um lote (nesse caso, o triager é invocado de novo só para esse lote).

## Fase 3 — Correção (sonar-fixer, Haiku)

1. Antes do primeiro lote, rode a linha de base com os comandos de verificação do cabeçalho do plano e anote o que já falhava. **Rode a suíte de testes duas vezes**: uma única rodada verde não distingue "verde" de "flaky", e um flake no meio da sessão faz o fixer reverter um lote bom. Passe ao fixer o que já falhava e os flakes conhecidos.
2. Para cada lote aprovado, em ordem e **um de cada vez**, invoque `sonar-fixer` com o caminho do plano e o número do lote.
   - Lotes marcados com `requer-sonnet: sim` ou com risco alto: invoque o `sonar-fixer` com o modelo trocado para `sonnet` (parâmetro `model` do Agent) desde o início.
3. A verificação de cada lote é **local**: testes, lint e build, mais os diagnósticos da extensão da IDE quando existirem. O servidor **não** é consultado entre lotes — os dados dele estão velhos por construção até o rescan da Fase 5.
4. Trate o retorno:
   - `OK`: revise rapidamente o diff do lote (`git diff`) para confirmar que ficou restrito às ocorrências. Se estiver certo, ofereça o commit pelo fluxo `/commit` com o tipo sugerido. Depois, siga para o próximo lote.
   - `ESCALAR`: invoque de novo o `sonar-fixer` para o mesmo lote com o modelo `sonnet`. Se o motivo for um plano ambíguo, peça antes ao triager para replanejar o lote.
   - `FALHOU`: confirme que o fixer reverteu as edições. Então tente o lote uma vez com `sonnet`; se falhar de novo, pare e apresente o erro ao usuário.
   - Ocorrências `divergentes`: o código mudou desde a análise do servidor. Junte-as e trate-as ao final, depois do rescan.
5. Nunca rode dois fixers em paralelo: os lotes podem tocar os mesmos arquivos, e a verificação precisa isolar cada lote.

## Fase 4 — Security hotspots

Apresente a seção de hotspots do plano, alimentada por `/api/hotspots/search`. Só gere um lote de correção para um hotspot (via triager) se o usuário concordar; esses lotes são sempre de risco alto e vão direto para o `sonar-fixer` com `sonnet`. A mudança de status do hotspot é feita pelo usuário, nunca por você — o relatório entrega o comando.

## Fase 5 — Rescan opcional e relatório

1. Ofereça o rescan: detecte o scanner do projeto (veja `references/webapi.md`), **mostre o comando e peça aprovação**, com o token por variável de ambiente. Rodando, espere a task do Compute Engine virar `SUCCESS` e releia issues e quality gate.
2. Salve `docs/sonarqube/SONAR-<data>-<projeto>.md` (criando o diretório se não existir) com:
   - perfil da conexão (URL, projeto, versão, edição/branch — **sem o token**) e escopo;
   - contagens e quality gate **antes e depois** (se houve rescan; senão, diga explicitamente que os números do servidor são anteriores às correções);
   - lotes aplicados, com o modelo que efetivamente corrigiu cada um (Haiku, ou Sonnet após escalada) e os issue keys;
   - falsos positivos com justificativa **e o comando de write-back pronto** para o usuário rodar;
   - itens fora de escopo, divergentes e pendências; referência ao plano `docs/sonarqube/LOTES-*.md`.
3. Apague o arquivo de config do curl do scratchpad.
4. As próximas sessões começam pelos relatórios em `docs/sonarqube/`: o triager os lê para não triar de novo falsos positivos já justificados.

## Regras da sessão

- Corrija a regra, não a métrica: nada de `// NOSONAR`, `@SuppressWarnings`, regras desativadas, exclusões (`sonar.exclusions`, `sonarlint.rules`) ou mudança de quality profile sem aprovação explícita.
- O token nunca entra no repositório nem numa linha de comando; use sempre o config do curl no scratchpad e apague-o no fim.
- A skill **não escreve no servidor**: nenhuma transição de issue, comentário ou mudança de status de hotspot. Esses comandos vão no relatório, para o usuário executar.
- `-k`/`--insecure` e qualquer relaxamento de TLS só com aprovação explícita do usuário.
- Nenhum código é editado antes de o usuário aprovar o plano de lotes.
- Um lote por vez; um lote só está concluído quando os testes, o lint e o build passam.
- Mudança de comportamento ou de API pública = pare e pergunte ao usuário.
- Os agentes nunca commitam; o commit é feito pela sessão principal via `/commit`, com aprovação do usuário.
