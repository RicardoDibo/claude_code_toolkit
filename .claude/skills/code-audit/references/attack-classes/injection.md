# Classe: injection

SQL, NoSQL, comando de sistema, template (SSTI), path traversal, XXE, LDAP/XPath, header injection.
Padrões de grep: `grep-patterns.md` § Injeção.

## Quando a unidade existe no ledger
Entry point que recebe dado externo **e** algum sink de consulta, shell, filesystem, template ou parser XML
alcançável a partir dele.

## O que caçar
1. **SQL:** string montada por concatenação, f-string, `format`, template literal ou `Sprintf` e passada a
   `execute`/`query`/`raw`. Atenção a partes que não aceitam parâmetro: nome de coluna em `ORDER BY`, nome de
   tabela, `LIMIT` montado, lista de `IN` montada à mão.
2. **NoSQL:** objeto da requisição passado direto como filtro (`find(req.body)`), operadores `$where`, `$regex`,
   `$ne` vindos do cliente.
3. **Comando:** `shell=True`, `exec`, `system`, `child_process.exec`, `sh -c` com interpolação; argumento que
   começa com `-` virando opção (argument injection) mesmo sem shell.
4. **Path traversal:** caminho montado com nome de arquivo vindo de fora sem normalizar e checar o prefixo;
   extração de zip/tar sem checar cada entrada (zip slip).
5. **Template (SSTI):** template **compilado** a partir de string do usuário (`render_template_string(input)`,
   `new Function`, `Template(input)`), não apenas variável renderizada num template fixo.
6. **XXE:** parser XML com entidades externas ou DTD habilitados recebendo XML de fora.
7. **Header/CRLF:** valor externo em header de resposta ou redirect sem remover `\r\n` (só se o framework não bloqueia).

## Seguro vs. inseguro
- Inseguro: `cur.execute(f"SELECT * FROM users WHERE name = '{name}'")` — Seguro: `cur.execute("SELECT * FROM users WHERE name = %s", (name,))`
- Inseguro: `` db.query(`... ORDER BY ${req.query.sort}`) `` — Seguro: `sort` validado contra allowlist de colunas antes de montar.
- Inseguro: `subprocess.run(f"convert {fname} out.png", shell=True)` — Seguro: `subprocess.run(["convert", "--", fname, "out.png"])`
- Inseguro: `open(os.path.join(BASE, name))` — Seguro: resolver o caminho real e exigir que comece com `BASE` resolvido.

## Perguntas de refutação (finding-verifier)
- O valor chega de fato ao sink sem parâmetro/escape? Siga o trace; um ORM ou query builder no meio costuma parametrizar.
- Existe validação (allowlist, tipo numérico, schema) entre a fonte e o sink? Onde (`arquivo:linha`)?
- A fonte é controlada pelo atacante ou é config/env/constante (exclusão nº 9)?
- O entry point é alcançável sem privilégio que já dá o mesmo poder (admin rodando comando no próprio servidor)?

## Não reportar
Exclusões nº 9, 15, 20 e 22 de `exclusions.md`. ORM com parâmetros, `execFile`/lista de argumentos sem shell,
caminho montado só a partir de constantes.

## CWE comuns
CWE-89 (SQL), CWE-943 (NoSQL), CWE-78 (comando), CWE-88 (argumento), CWE-22 (path), CWE-1336 (SSTI), CWE-611 (XXE), CWE-113 (CRLF).

## Correção típica
Parametrizar no padrão do projeto; allowlist para identificadores; lista de argumentos sem shell e `--` antes
de valores; normalizar e checar prefixo de caminho; desligar DTD/entidades externas no parser.
