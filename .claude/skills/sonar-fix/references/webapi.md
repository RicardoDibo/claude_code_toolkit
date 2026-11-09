# Referência — Web API do SonarQube como fonte de achados

Fonte **canônica** dos achados da skill `sonar-fix`. Vale para SonarQube Community Build, SonarQube Server (9.x, 10.x, 2025.x) e SonarQube Cloud. Nada aqui é assumido pela versão: o que varia é **descoberto em runtime** (seção "Descoberta de capacidades").

## 1. Conexão e autenticação

| O quê | Chamada | Observação |
|-------|---------|------------|
| Servidor está no ar | `GET /api/system/status` | não exige auth; devolve `{"id":...,"version":"...","status":"UP"}` |
| Versão exata | `GET /api/server/version` | texto puro, sem JSON |
| Projeto existe | `GET /api/components/search?qualifiers=TRK&ps=100` | valida ou descobre o project key |

- **Token:** o ideal é um **User token** (UI: *My Account → Security*), que herda as permissões do usuário; para a triagem ele precisa de **Browse** e **See Source Code** no projeto.
- **O prefixo diz o tipo, e o tipo limita o que você consegue ler.** Confira o prefixo (sem imprimir o resto) e registre no perfil:

  | Prefixo | Tipo | Na prática |
  |---------|------|------------|
  | `squ_` | user token | leitura completa; é o único que serve para write-back |
  | `sqa_` | global analysis token | lê issues, rules, measures, quality gate e branches, mas dá **403** em `/api/project_analyses/search`; write-back falha |
  | `sqp_` | project analysis token | igual ao `sqa_`, restrito a um projeto |

  Com token de análise a sessão roda normalmente: só não tente `project_analyses/search` (use `analysisDate` de `project_branches/list` para a data da última análise) nem execute transições — documente-as no relatório, que é o padrão da skill de todo modo.
- **Esquema preferido:** header `Authorization: Bearer <token>` (SonarQube 10.0+ e Community Build).
- **Fallback:** se a resposta for `401` com Bearer, o servidor é antigo — repita com basic auth usando o token como usuário e senha vazia (`user = "<token>:"` no config do curl). Registre no perfil qual esquema funcionou e use o mesmo até o fim da sessão.
- A resposta traz o header `SonarQube-Authentication-Token-Expiration`: se estiver vencendo, avise o usuário.

## 2. Manuseio do token (regra dura)

O token **nunca** entra no repositório e **nunca** aparece numa linha de comando — a linha fica no transcript e no histórico do shell.

Grave um arquivo de config do curl no **scratchpad da sessão** (fora do projeto), uma opção por linha:

```
header = "Authorization: Bearer SEU_TOKEN"
silent
show-error
```

E chame sempre com `--config`:

```bash
curl --config "$CFG" "$SONAR_URL/api/server/version"
```

- **Windows/PowerShell:** use `curl.exe`. No PowerShell 5.1 `curl` é alias de `Invoke-WebRequest` e os flags do curl não valem. Pelo tool Bash, `curl` normal.
- **`WebFetch` não alcança `localhost`** (nem hosts sem ponto): todo acesso a um servidor local é por `curl` via Bash/PowerShell.
- Ao fim da sessão, apague o arquivo de config.

## 3. Descoberta de capacidades (o mecanismo "qualquer versão")

```
GET /api/webservices/list
```

Devolve, **para aquela instância**, todos os endpoints com seus parâmetros e valores aceitos. Antes de usar um parâmetro que mudou entre versões, confirme ali — não confie na memória. A mesma informação em HTML fica em `http://<host>/web_api`.

Parâmetros que **precisam** ser confirmados antes do primeiro uso:

| Assunto | Legado | Novo |
|---------|--------|------|
| componente | `componentKeys` | `components` |
| taxonomia | `types`, `severities` | `impactSeverities`, `impactSoftwareQualities` |
| status | `statuses` (dep. 10.4) + `resolutions` | `issueStatuses` {`OPEN`, `CONFIRMED`, `FALSE_POSITIVE`, `ACCEPTED`, `FIXED`, `IN_SANDBOX`} |
| new code | `sinceLeakPeriod` | `inNewCodePeriod` |

Em versões recentes (verificado no Community Build 26.11) `componentKeys` **já não aparece** na lista de parâmetros, embora ainda responda: use `components`. O endpoint **não** valida parâmetro desconhecido — ele é ignorado em silêncio, e o resultado vira "o projeto todo" em vez do escopo que você pediu. Depois da primeira chamada, confira sempre o `total` contra `/api/measures/component`.

Armadilha conhecida: em algumas versões `resolutions=` com múltiplos valores devolve `total=0` silenciosamente. Prefira `issueStatuses` quando existir.

**Branches — não sonde com `&branch=`, liste:**

```
GET /api/project_branches/list?project=<key>
```

A sonda por código HTTP é inútil: o parâmetro `branch` é aceito e **filtra** mesmo em Community Build, e uma branch que não existe devolve **`total=0` sem erro algum** — o modo de falha mais perigoso, porque parece "projeto limpo". Passe apenas um nome que apareça em `project_branches/list` (no Community costuma ser só a `main`, com `isMain: true`), ou omita o parâmetro. Registre no perfil quais branches existem e qual é a principal.

**Hotspots:** `/api/hotspots/*` está **deprecado desde 2026.4** e usa `project=`, não `projectKey=` — passar `projectKey` dá `400`. Os hotspots estão migrando para `/api/issues/search` (filtre por `impactSoftwareQualities=SECURITY`). Confirme em `webservices/list` se as ações de `hotspots` ainda existem na sua versão antes de usá-las.

## 4. Catálogo de endpoints

```bash
# issues (paginar: ps máximo 500; facets dão o panorama numa chamada)
GET /api/issues/search?components=<key>&resolved=false&ps=500&p=<n>&facets=rules,files,types,impactSeverities
# branches existentes (antes de usar branch=)
GET /api/project_branches/list?project=<key>
# hotspots (deprecado em 2026.4; note "project", não "projectKey")
GET /api/hotspots/search?project=<key>&status=TO_REVIEW&ps=500&p=<n>
# descrição da regra
GET /api/rules/show?key=<repo>:<Sxxxx>
# quality gate
GET /api/qualitygates/project_status?projectKey=<key>
# resumo
GET /api/measures/component?component=<key>&metricKeys=bugs,vulnerabilities,code_smells,security_hotspots,coverage
# última análise
GET /api/project_analyses/search?project=<key>&ps=1
# fonte no servidor (só se o arquivo local divergir)
GET /api/sources/raw?key=<componentKey>
# fila de análise (depois do rescan)
GET /api/ce/activity?component=<key>&ps=1
GET /api/ce/task?id=<taskId>
```

- **Teto de paginação:** `/api/issues/search` não devolve além de ~10.000 issues. Se `total` chegar perto disso, estreite o escopo: diretório (`componentKeys=<key>:src/api`), `rules=`, `languages=`, new code period ou severidade.
- **`/api/rules/show` é preferido** a `rules.sonarsource.com`: reflete o quality profile da sua instância e funciona sem internet. A descrição vem em `htmlDesc` (versões antigas) ou em `descriptionSections[]` (novas: `introduction`, `root_cause`, `how_to_fix`, `resources`). O site público fica como fallback, via `WebFetch` em `https://rules.sonarsource.com/<linguagem>/RSPEC-<número>/`.

## 5. Mapa dos campos para o código local

- `component` = `<projectKey>:<caminho/relativo.ext>`. O caminho local sai removendo o prefixo `<projectKey>:` — **confirme com `Read`/`Glob` que o arquivo existe**: em monorepo e multi-módulo o prefixo inclui o módulo (`meu-app:backend:src/x.ts`) e o caminho bruto não bate.
- `key`: o issue key (ex.: `AZ8kP...`). Carregue-o no plano e no relatório; é o que liga a correção à issue do servidor depois do rescan.
- `line` e `textRange` (`startLine`, `endLine`, `startOffset`, `endOffset`): **base 1**, ao contrário do diagnóstico do VS Code (base 0).
- `rule`, `message`, `effort`, `tags`, `author`, `creationDate`.
- Taxonomia: `type` + `severity` (legado: `BUG`/`VULNERABILITY`/`CODE_SMELL`, `BLOCKER`…`INFO`) e `impacts[]` + `cleanCodeAttribute` (10.2+). Use o que a instância devolver; não invente o campo ausente.
- `flows[].locations[]`: o caminho fonte→sink das issues de taint. Material direto para o campo "Padrão de correção" do lote.
- `status`/`resolution` (ou `issueStatus`): descarte o que já está `FALSE_POSITIVE`/`ACCEPTED`/`WONTFIX` — já foi triado por alguém.

### Frescor: verifique por conteúdo, não por data

Comparar `analysisDate` com `git log -1` **não** responde se os dados servem: um scan rodado de um working tree sujo, ou sem `sonar.branch.name`, fica com data anterior aos commits e com o rótulo de branch errado, e ainda assim descreve exatamente o código que está no disco. O contrário também ocorre.

O teste que vale é amostrar `textRange` contra os arquivos locais: para cada issue, corte `linha[startLine-1].slice(startOffset, endOffset)` e veja se o trecho faz sentido para a regra (`Math.random()` para `S2245`, `sort` para `S2871`). Faça isso com todas as issues do escopo antes de triar, e classifique:

- **trecho coerente** → dado fresco para aquela ocorrência;
- **trecho vazio ou sem relação** → deslocado: a ocorrência é válida, mas localize-a pelo trecho/pela regra, não pela linha;
- **linha além do fim do arquivo** ou **arquivo inexistente** → fora de escopo, a análise é de outra versão do código.

Uma taxa alta de deslocamento é o sinal de que vale rodar o rescan antes de triar, e não o relógio.

## 6. Rescan (reanálise)

O servidor **não** reanalisa sozinho: depois das correções os dados ficam velhos até o scanner rodar de novo. Detecte o scanner do projeto nesta ordem:

| Pista no projeto | Comando |
|---|---|
| `sonar-project.properties` + CLI `sonar-scanner` | `sonar-scanner -Dsonar.host.url=<url>` |
| `pom.xml` | `mvn -B sonar:sonar -Dsonar.host.url=<url>` |
| `build.gradle` com o plugin `org.sonarqube` | `./gradlew sonar -Dsonar.host.url=<url>` |
| `sonarqube-scanner` no `package.json` | o script npm correspondente |
| nenhum dos acima | `docker run --rm -v "$PWD:/usr/src" sonarsource/sonar-scanner-cli` |

- O token vai por **variável de ambiente** (`SONAR_TOKEN`), nunca em `-Dsonar.token=` na linha de comando.
- Container local: de dentro do container do scanner, `localhost` é o próprio container. Use `--network host` ou o nome do serviço na rede do compose.
- O comando é **mostrado e aprovado** pelo usuário antes de rodar.
- Depois: `/api/ce/activity?component=<key>&ps=1` → pegue o `id` da task e espere `/api/ce/task?id=<id>` virar `SUCCESS` (se `FAILED`, mostre `errorMessage`). Só então releia issues e quality gate.

## 7. Write-back (documentado; a skill não executa)

A skill **não** altera estado no servidor. O relatório entrega o comando pronto para o usuário rodar:

```bash
# marcar falso positivo (transition: falsepositive | accept | confirm | resolve | reopen)
curl --config "$CFG" -X POST "$SONAR_URL/api/issues/do_transition" \
  -d "issue=<issueKey>" -d "transition=falsepositive"
# justificativa
curl --config "$CFG" -X POST "$SONAR_URL/api/issues/add_comment" \
  -d "issue=<issueKey>" -d "text=<justificativa>"
# hotspot
curl --config "$CFG" -X POST "$SONAR_URL/api/hotspots/change_status" \
  -d "hotspot=<key>" -d "status=REVIEWED" -d "resolution=SAFE"
```

`wontfix` foi renomeado para `accept` no SonarQube 10.4 — confira em `webservices/list` qual o nome na sua versão. Essas transições exigem a permissão *Administer Issues* no projeto.

## 8. Troubleshooting

| Sintoma | Causa provável |
|---|---|
| `401` | token inválido, expirado, ou servidor antigo que não aceita Bearer (tente basic auth) |
| `403` | o usuário do token não tem Browse/See Source Code (ou Administer Issues, no write-back) |
| `404` no projeto | project key errado → liste com `/api/components/search?qualifiers=TRK` |
| `400` em `branch` | edição sem multi-branch (Community) → remova o parâmetro |
| `total: 0` com projeto cheio | filtro de status/resolution inválido na sua versão, escopo errado, ou tudo já resolvido — cruze com `/api/measures/component` |
| medidas vazias | o projeto nunca foi analisado → rode o scanner primeiro |
| conexão recusada | container parado, porta não publicada, ou `localhost` visto de dentro de outro container/WSL |
| erro de certificado | HTTPS com CA própria → `--cacert <arquivo>`. `-k`/`--insecure` **só** com aprovação explícita do usuário |

## 9. Configurações que exigem aprovação para alterar

`sonar-project.properties` · `sonar.exclusions` / `sonar.coverage.exclusions` · quality profile e quality gate no servidor · qualquer transição de issue ou hotspot.
