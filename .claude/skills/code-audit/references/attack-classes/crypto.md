# Classe: crypto

Algoritmo fraco no contexto errado, aleatoriedade previsível, validação de certificado desligada, gestão de chave,
hash de senha. Padrões de grep: `grep-patterns.md` § Cripto.

## Quando a unidade existe no ledger
Código que gera token/ID secreto, guarda senha, cifra ou assina dado, ou faz HTTPS/TLS de saída.

## O que caçar
1. **Hash de senha:** MD5, SHA-1, SHA-256 simples ou sem salt para senha. Esperado: bcrypt, scrypt, argon2, PBKDF2 com custo adequado.
2. **Aleatoriedade:** `Math.random`, `random.random`, `rand.Intn`, `new Random()` gerando token de sessão, reset de
   senha, código de convite, API key. Esperado: `secrets`, `crypto.randomBytes`/`randomUUID`, `crypto/rand`, `SecureRandom`.
3. **Cifra:** DES, 3DES, RC4, modo ECB, IV fixo ou reutilizado, chave derivada de senha sem KDF, cifra sem
   autenticação onde há integridade em jogo (CBC sem MAC).
4. **TLS:** `verify=False`, `rejectUnauthorized: false`, `InsecureSkipVerify: true`, `CERT_NONE`, `TrustAllCerts`
   em código de runtime (não em teste).
5. **Comparação de segredo:** `==` em HMAC/token quando há canal de tempo alcançável (só se for prático, exclusão nº 13).
6. **Chave:** chave de cifra/assinatura hardcoded (vira também `secrets`), chave curta, mesma chave para fins diferentes.

## Seguro vs. inseguro
- Inseguro: `hashlib.sha256(password.encode()).hexdigest()` — Seguro: `bcrypt.hashpw(password, bcrypt.gensalt())`
- Inseguro: `token = Math.random().toString(36)` — Seguro: `crypto.randomBytes(32).toString("hex")`
- Inseguro: `requests.get(url, verify=False)` — Seguro: verificação padrão, com CA customizada se necessário.

## Perguntas de refutação
- Para que o hash/aleatório é usado? Checksum, cache key, ETag, sharding e IDs não secretos não são achado (exclusão nº 16).
- O `verify=False` está em código de runtime ou em script/teste local?
- A biblioteca já usa um padrão seguro por baixo (ex.: framework que gera o token de sessão)?

## Não reportar
Exclusões nº 13, 15 e 16. Ausência de TLS ou HSTS na aplicação (é deploy, não código), salvo se o spec exigir.

## CWE comuns
CWE-916 (hash de senha fraco), CWE-327 (algoritmo quebrado), CWE-330/CWE-338 (aleatoriedade), CWE-295 (certificado),
CWE-329 (IV previsível), CWE-321 (chave hardcoded), CWE-208 (comparação por tempo).

## Correção típica
Usar a primitiva de alto nível que o ecossistema recomenda e que o projeto já usa em outro lugar; não inventar
esquema próprio; migração de hash de senha com rehash no próximo login (anote "toca contrato" se mudar formato persistido).
