# Classe: secrets

Segredo hardcoded, segredo versionado, segredo em log, e credencial que vaza por serialização automática.
Ferramentas: gitleaks/trufflehog (`tools.md` nº 1–4). Padrões de grep: `grep-patterns.md` § Segredos.

## Regra de ouro
Nunca escreva o valor em claro em nenhum lugar (chat, candidato, relatório, log): 4 primeiros caracteres + `****`.
Nunca teste se a credencial funciona (isso é a verificação ativa do trufflehog, só com pedido explícito do usuário).

## O que caçar
1. **Hardcoded em código de runtime:** chaves de provedor (formatos em `grep-patterns.md`), senhas de banco,
   segredos de assinatura (JWT, cookie, HMAC), chaves privadas.
2. **Arquivo versionado:** `.env`, `*.pem`, `*.key`, `credentials.json` rastreados pelo git (`git ls-files`).
3. **Histórico (só `deep`):** segredo removido do arquivo mas presente em commits anteriores (gitleaks `git`).
4. **Segredo em log:** log de request/response inteiro, de headers (`Authorization`, `Cookie`), de objetos de
   configuração, de exceções que carregam a connection string.
5. **Vazamento por serialização** (Sentry, secret-serialization): objeto que guarda a credencial (client, config,
   model) e é serializado sem filtro — `repr`/`__str__` de dataclass, `toJSON`, `model_dump()`, span de tracing,
   erro devolvido ao cliente. Procure o portador da credencial e, depois, os sinks que o serializam.
6. **Default inseguro:** segredo com valor padrão no código (`os.getenv("SECRET_KEY", "dev-secret")`) usado em produção.
7. **Segredo no front-end:** chave secreta (não a publicável) embutida no bundle do cliente ou em variável
   `NEXT_PUBLIC_`/`VITE_`/`REACT_APP_`.

## Perguntas de refutação
- É placeholder, exemplo ou valor de teste (exclusão nº 18 e ajustes por caminho)? O formato é de credencial real do provedor?
- É chave **publicável** por desenho (Stripe `pk_`, Firebase config, chave pública)? Então não é achado.
- O arquivo é de runtime ou só de teste/documentação?
- Em log: o valor logado é de fato o segredo, ou um ID/URL sem credencial?

## Severidade
- Segredo de produção em código de runtime ou versionado: `crítica`.
- Segredo de teste com formato real de provedor: `média` (pode dar acesso a ambiente de teste).
- Segredo em log de produção: `alta`.
- Default inseguro usado só em dev, com produção exigindo a variável: não é achado (falha fechado).

## Não reportar
Exclusões nº 2 (segredo em disco já protegido), 9, 17 e 18.

## CWE comuns
CWE-798 (credencial hardcoded), CWE-312/CWE-532 (segredo em claro / em log), CWE-200 (exposição), CWE-1188 (default inseguro).

## Correção típica
Mover para variável de ambiente ou cofre já usado pelo projeto; **rotacionar** a credencial (remover do arquivo
não limpa o histórico); adicionar o arquivo ao `.gitignore`; filtrar campos sensíveis na serialização/log.
A rotação e a limpeza de histórico são ações do usuário: o relatório as recomenda, o agente não as executa.
