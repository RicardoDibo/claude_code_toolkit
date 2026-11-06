# Formatos dos artefatos do /code-audit

Todos ficam em `audit/<AAAA-MM-DD>-<escopo>/`:

```
audit/
├── exclusions.local.md          ← opcional, mantido pelo usuário (prevalece sobre exclusions.md)
└── <AAAA-MM-DD>-<escopo>/
    ├── architecture.md          ← audit-recon (Fase 1)
    ├── coverage-ledger.md       ← audit-recon cria; o orquestrador atualiza os status com os retornos dos hunters (Fases 2–3)
    ├── candidates-scanners.md   ← security-hunter modo scanners (Fase 2)
    ├── candidates-<classe>.md   ← security-hunter modo caça/critic (Fase 3)
    ├── raw/                     ← saída bruta dos scanners (sugerir no .gitignore)
    └── AUDIT.md                 ← orquestrador (Fase 6); Decisão e Resultado depois
```

Em todos eles: segredo só redigido (4 primeiros caracteres + `****`), toda afirmação com `arquivo:linha`.

## 1. `architecture.md` (audit-recon)

```markdown
# Arquitetura — <alvo> — <data>
Perfil: <perfil> | Arquivos de runtime: <n> | Linguagens: <lista> | Frameworks: <lista>

## Stack e execução
- Linguagens, frameworks, gerenciadores de pacote, manifests e lockfiles (caminhos).
- Como roda: servidor HTTP, CLI, worker, função serverless, app desktop/mobile, biblioteca.
- Código de runtime: <pastas>. Fora do runtime: testes <pastas>, CI <pastas>, scripts <pastas>, vendor <pastas>.

## Entry points
| ID | Tipo | Onde | Autenticado? | Observação |
|----|------|------|--------------|------------|
| E1 | HTTP `POST /login` | `src/routes/auth.ts:12` | não | |

## Principais e autorização
- Quem são os atores (anônimo, usuário, admin, serviço) e onde a identidade é estabelecida (`arquivo:linha`).
- Onde a autorização é aplicada (middleware global, decorator, checagem na rota, no serviço) e o padrão do projeto.

## Fontes de dado não confiável
Corpo/params/headers/cookies, uploads, webhooks, filas, arquivos lidos de fora, respostas de APIs externas, saída de LLM.

## Sinks sensíveis
SQL/NoSQL, shell/subprocess, filesystem (caminho montado), HTTP de saída, templates/HTML, desserialização,
eval/reflexão, cripto, logs, prompts com ferramentas — cada um com `arquivo:linha` representativo.

## Fronteiras de confiança
| ID | Fronteira | De → Para | Controle observado |
|----|-----------|-----------|--------------------|
| B1 | internet → API | cliente → `src/server.ts` | middleware de auth em `src/middleware/auth.ts:8` |

## Padrões de segurança do projeto
Sanitizadores, ORM/query builder, helpers de authz, gestão de segredos (env, vault), configuração de CORS/CSRF/cookies.
Desvio desses padrões é onde a caça começa.

## Intenção declarada
Resumo das regras de "Validações e Segurança" dos spec.md, do SECURITY.md/SECURITY_MODEL.md e das exclusões locais.

## Ferramentas
| Categoria | Ferramenta | Situação | Comando proposto | Rede |
|-----------|-----------|----------|------------------|------|

## Premissas
O que não foi possível confirmar no código (ex.: "assumo que o proxy termina TLS"), cada uma marcada como premissa.
```

## 2. `coverage-ledger.md` (audit-recon cria; orquestrador atualiza)

```markdown
# Coverage ledger — <alvo> — <data>

| ID | Superfície (entry point / componente) | Fronteira | Classe | Dono | Status | Candidatos / motivo |
|----|---------------------------------------|-----------|--------|------|--------|---------------------|
| U1 | E1 `POST /login` | B1 | injection | caça injection | coberta | C4 |
| U2 | E1 `POST /login` | B1 | authz | caça authz | sem candidato | |
| U3 | `Dockerfile` | build | iac | scanners | não avaliada | trivy ausente e sem rede |
```

- Classes válidas: `injection`, `authz`, `secrets`, `crypto`, `deserialization`, `ssrf`, `supply-chain`, `iac`, `llm` (uma `attack-classes/<classe>.md` para cada).
- `prompt-injection` não é classe do ledger: é a categoria do achado quando qualquer agente encontra, de passagem, texto do repositório dirigido a agentes. O orquestrador junta essas citações dos retornos e as manda verificar como candidatos `PI-n`.
- Status: `pendente` (criação) → `coberta` | `sem candidato` | `não avaliada — motivo`. Nenhuma unidade termina `pendente`.
- Uma unidade só existe se a classe se aplica à superfície (não crie `ssrf` para uma rota que não faz HTTP de saída).

## 3. `candidates-*.md` (security-hunter)

```markdown
# Candidatos — <scanners | classe> — <data>

## <prefixo>-1 — <título curto>
**Origem:** semgrep `python.lang.security.audit.formatted-sql-query` | caça manual | critic
**Classe / CWE provável:** injection / CWE-89
**Unidade do ledger:** U1
**Trace:**
- entrypoint: `src/routes/users.py:12` — `GET /users?sort=`
- propagação: `src/services/users.py:40` — repassa `sort` sem validação
- sink: `src/db/query.py:88` — f-string em `ORDER BY`
**Hipótese:** um usuário anônimo controla `sort` e ele chega ao SQL sem parâmetro nem allowlist.
**Severidade provável:** alta
**Pontos a verificar:** existe validação de `sort` em middleware? A rota exige login?
```

Prefixos: `SC` (scanners), `INJ`, `AUTHZ`, `SEC`, `CRY`, `DES`, `SSRF`, `SUP`, `IAC`, `LLM`, `PI`, `CRT` (critic).
Pista de scanner que o hunter de classe confirmou como candidato é reescrita no `candidates-<classe>.md`
com o trace completo e a origem do scanner; a pista original fica como está.

## 4. Retorno do `finding-verifier` (um bloco por candidato)

```markdown
### <ID do candidato>
**Veredito:** CONFIRMED | PLAUSIBLE | descartado
**Confiança:** <1–10>
**Alegação reescrita:** atacante <quem>, sem <o quê>, consegue <impacto> via `arquivo:linha`.
**Gates:** alcançabilidade ✔ | controle do atacante ✔ | impacto real ✔ | ambiente/mitigação ✔ | exclusões ✔
**Bloqueio (só PLAUSIBLE):** o que falta confirmar e quem consegue confirmar.
**Motivo do descarte (só descartado):** gate que falhou ou nº da exclusão.
**Severidade final:** crítica | alta | média | baixa
**Fingerprint:** <classe>:<arquivo>:<função ou símbolo>:<parâmetro ou chave>
(campos do achado, seção 5, quando CONFIRMED ou PLAUSIBLE)
```

CONFIRMED exige confiança ≥ 8. Confiança 5–7 com bloqueio nomeado → PLAUSIBLE. Abaixo disso → descartado.

## 5. `AUDIT.md` (orquestrador)

```markdown
# Auditoria — <alvo> — <perfil> — <data>
Ferramentas: gitleaks 8.21.2 ✔ | semgrep ✘ (Grep) | osv-scanner 2.0.1 ✔ | trivy ✘ (IaC não avaliado)
Rede aprovada: <comandos [rede] aprovados | nenhuma>
Ledger: <n> unidades — <n> cobertas / <n> sem candidato / <n> não avaliadas
Candidatos: <n> | Descartados: <n> | CONFIRMED: <n> | PLAUSIBLE: <n> | Cortados pelo máximo: <n>
Comandos de verificação do projeto: <testes> / <lint> / <build>
Qualidade: <quality/QUALITY-...md | não rodou (--so-seg)>

## Resumo
| ID | Sev. | Classe | Arquivo:linha | Achado (≤60 caracteres) | Veredito | Conf. | Toca contrato | Decisão |
|----|------|--------|---------------|-------------------------|----------|-------|---------------|---------|
| S1 | crítica | secrets | `src/config.ts:4` | Chave de API de produção commitada | CONFIRMED | 10 | não | ☐ aplicar ☐ dispensar |

## Segurança

## S1 — <título curto>
**Severidade / classe / CWE / veredito / confiança:** alta / injection / CWE-89 / CONFIRMED / 9
**Trace:**
- entrypoint: `src/routes/users.py:12` — `GET /users?sort=`
- propagação: `src/services/users.py:40`
- sink: `src/db/query.py:88`
**Outras ocorrências:** `arquivo:linha`, … (mesma causa raiz)
**Pré-condição do atacante:** usuário anônimo.
**Cenário de exploração:** o que o atacante faz e o que obtém, descrito (sem payload operacional contra sistema real).
**Evidência:** trecho do código, pista do scanner (`raw/semgrep.json`, regra X) ou regra do spec.md violada.
**Correção recomendada:** o que mudar e onde, no padrão de segurança que o projeto já usa. **Não fazer:** armadilhas.
**Teste a adicionar:** caso que falha hoje e passa depois da correção.
**Toca contrato:** sim | não
**Tipo de commit sugerido:** fix | build(deps)
**Fingerprint:** injection:src/db/query.py:build_order_by:sort
**Resultado:** _(preenchido pelo feature-implementer: aplicado | não aplicado — motivo)_

## Qualidade
Resumo da tabela do code-improver (IDs `Q…`, severidade, achado) e o link para `quality/QUALITY-...md`,
onde ficam os detalhes, a Decisão e o Resultado dos `Q…`.

## Precisa de verificação
| ID | Achado | Bloqueio | Quem pode confirmar |
(os PLAUSIBLE também aparecem no Resumo; aqui fica o que falta para fechar cada um)

## Lacunas de cobertura
| Unidade / categoria | Motivo | Como cobrir |
| SCA | osv-scanner ausente e sem rede | instalar osv-scanner ou aprovar rede na próxima rodada |

## Descartados
| Motivo (gate ou exclusão nº) | Quantidade | Exemplos (ID do candidato) |
```

**Achados de dependência** (`supply-chain` vindo de SCA): um achado por pacote, com versão instalada, versão
corrigida, advisories, direta/transitiva, runtime/dev e o **comando de atualização para o usuário rodar**;
tipo de commit `build(deps)`. O feature-implementer não os aplica.

**Rodadas seguintes:** nova pasta com data própria; o cabeçalho cita a rodada anterior, e achados com o mesmo
fingerprint de um `dispensar` anterior não entram (listados em Descartados como "dispensado na rodada X").
