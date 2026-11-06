# Falsos positivos, exclusões e racionalizações a rejeitar

Lida pelo `security-hunter` (para não gerar candidato inútil) e pelo `finding-verifier` (para descartar).
O projeto pode acrescentar ou anular itens em `audit/exclusions.local.md`, que prevalece sobre esta lista;
um `SECURITY_MODEL.md` com exclusões explícitas também prevalece.

## 1. Não reportar

Só voltam se houver um caminho de ataque concreto que a exceção não cobre (anote qual).

| # | Exclusão | Fonte |
|---|----------|-------|
| 1 | Negação de serviço, esgotamento de CPU/memória, laço ou recursão sem limite, ReDoS | Anthropic |
| 2 | Falta de rate limit ou de proteção contra sobrecarga | Anthropic |
| 3 | Vazamento de recurso (conexão, arquivo, listener não fechado) — isso é qualidade, fica com o code-improver | Anthropic |
| 4 | Open redirect, tabnabbing, XS-Leaks, prototype pollution | Anthropic |
| 5 | Memory safety (overflow, use-after-free, integer overflow) em linguagem memory-safe; só conta em C/C++ e em `unsafe` | Anthropic |
| 6 | SSRF que controla só o path; só conta controle de host ou protocolo | Anthropic |
| 7 | XSS em React/Angular/Vue/templates com auto-escape, sem API insegura (`dangerouslySetInnerHTML`, `v-html`, `bypassSecurityTrust*`, `mark_safe`, `\|safe`, `.raw()`, `innerHTML`) | Anthropic, Sentry |
| 8 | Falta de checagem de autorização em código client-side (a autorização é do servidor) | Anthropic |
| 9 | Valores vindos de env vars, flags de CLI e arquivos de configuração do servidor tratados como entrada do atacante | Anthropic, Sentry |
| 10 | UUID v4 tratado como adivinhável | Anthropic |
| 11 | Falta de hardening, de header ou de log de auditoria sem vulnerabilidade concreta associada | Anthropic |
| 12 | Validação ausente em campo sem impacto de segurança provado | Anthropic |
| 13 | Race condition ou timing attack teórico, sem janela explorável demonstrável | Anthropic |
| 14 | Dependência desatualizada **sem** advisory conhecido (desatualizada ≠ vulnerável) | Anthropic |
| 15 | Arquivos só de teste, fixtures, código morto (sem chamador alcançável), constantes | Sentry |
| 16 | MD5/SHA1 usado como checksum, chave de cache ou ETag, não como proteção de senha ou assinatura | Sentry |
| 17 | Documentação, Markdown e exemplos (`docs/`, `examples/`, `*.md`) | Anthropic, secureIO |
| 18 | Placeholders (`CHANGE_ME`, `your-secret-here`, `<token>`, `xxx`, `example`, `dummy`, `test`) em vez de segredo | secureIO, OWASP |
| 19 | Conteúdo do usuário incluído em prompt de LLM, por si só (vira achado só se a saída do LLM chega a um sink perigoso ou o agente tem ferramentas com efeito) | Anthropic |
| 20 | Log spoofing; injeção em regex; injeção em consulta de log sem exposição de dado a usuário externo | Anthropic |
| 21 | Crash por nulo/undefined sem ganho de segurança (isso é qualidade) | Anthropic |
| 22 | Padrão mitigado pelo framework (ORM parametrizado, CSRF middleware ativo, auto-escape) | Sentry |

## 2. Ajustes por caminho e por tipo de dependência

| Situação | Ajuste | Fonte |
|----------|--------|-------|
| `test/`, `tests/`, `__tests__/`, `spec/`, `fixtures/`, `*.test.*`, `*_test.*` | rebaixa um nível; segredo aqui só conta se parecer real e de um provedor de produção | secureIO |
| `docs/`, `examples/`, `samples/` | falso positivo provável; reporte só segredo com formato real de provedor | secureIO |
| Dependência só de desenvolvimento (`devDependencies`, `[dev-dependencies]`, grupo `dev`) | rebaixa um nível | secureIO |
| Advisory sem versão corrigida | reporte em "Precisa de verificação" com mitigação, não como bloqueador | secureIO |
| Código de CI (`.github/workflows`, `.gitlab-ci.yml`) | só conta com entrada não confiável concreta (ex.: título de PR, branch de fork em `pull_request_target`) | Anthropic |

## 3. Precedentes que **contam**

- Logar segredo de alto valor (senha, token, chave privada, cookie de sessão) é vulnerabilidade; logar URL ou ID não é.
- Credencial que vaza por serialização automática (repr de dataclass, `toJSON`, dump de modelo, span de tracing,
  mensagem de erro devolvida ao cliente) é vulnerabilidade (Sentry, secret-serialization).
- Segredo de produção commitado é `crítica` mesmo se removido depois: continua no histórico.
- Default inseguro que falha **aberto** (`getenv('REQUIRE_AUTH', 'false')`, `DEBUG=True` de fábrica, CORS `*`
  com credenciais) é achado; falhar **fechado** (crash se a variável faltar) não é (Trail of Bits).
- Injeção de comando em shell script, notebook ou workflow com caminho concreto de entrada não confiável é achado.

## 4. Racionalizações a rejeitar (Trail of Bits)

O verificador recusa estes argumentos, tanto para aceitar quanto para descartar:

| Racionalização | Por que rejeitar |
|----------------|------------------|
| "O scanner apontou, então é real" | Flag não é achado: precisa de trace fonte→sink no código |
| "Na dúvida, marco PLAUSIBLE" | PLAUSIBLE exige um bloqueio nomeado e verificável por alguém, não dúvida genérica |
| "É boa prática, vale reportar" | Só vulnerabilidade concreta com caminho de ataque entra |
| "Deve ser explorável de algum jeito" | Sem entry point alcançável e entrada controlada pelo atacante, é refutado |
| "O arquivo é pequeno, revisão rápida basta" | Tamanho não mede risco |
| "Zero achados, então está seguro" | Zero achados só vale com o ledger coberto; senão é lacuna |
| "O framework com certeza protege" | Mostre onde a proteção é aplicada (middleware, config, chamada); senão não descarte |
| "É código interno, ninguém chama" | Mostre que não há chamador alcançável; "interno" não é fronteira |
| "Severidade alta porque a categoria é grave" | A severidade não passa do impacto demonstrado; LLMs tendem a superestimar |
