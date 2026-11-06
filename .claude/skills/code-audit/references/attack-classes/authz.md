# Classe: authz

Autenticação e autorização: rota sem guarda, autorização na camada errada, IDOR/BOLA, escalada de privilégio,
sessão, JWT, CSRF, mass assignment. Padrões de grep: `grep-patterns.md` § Autorização e § Cripto (JWT).

## Quando a unidade existe no ledger
Toda superfície que expõe dado ou ação de um usuário/tenant, ou ação administrativa.

## O que caçar
1. **Rota sem guarda:** compare a lista de rotas com a aplicação do middleware/decorator de auth. A pista forte é a
   rota sem guarda no meio de rotas guardadas, ou a guarda registrada **depois** da rota (ordem de middleware).
2. **IDOR/BOLA:** handler que busca recurso por ID vindo da requisição sem filtrar pelo dono/tenant da sessão.
3. **Autorização por função:** ação de admin protegida só por autenticação, ou papel lido do corpo/headers/JWT não assinado.
4. **Mass assignment:** objeto da requisição gravado inteiro no modelo (`User(**req.json)`, `Object.assign(user, req.body)`)
   quando o modelo tem campos como `role`, `is_admin`, `owner_id`, `balance`.
5. **Sessão:** cookie de sessão sem `HttpOnly`/`Secure`/`SameSite` **e** com impacto concreto; ID de sessão não
   rotacionado no login; logout que não invalida no servidor.
6. **JWT:** verificação sem assinatura, `alg: none` aceito, chave fraca ou hardcoded, `decode` em vez de `verify`,
   claims de expiração ignoradas.
7. **CSRF:** ação que muda estado por cookie de sessão, sem token CSRF nem `SameSite`, em framework que não protege por padrão.
8. **Reset de senha / convite:** token previsível, sem expiração, reutilizável, ou vazando no log/URL de terceiros.
9. **Divergência do spec:** regra de "Validações e Segurança" do spec.md (ex.: "só o dono edita") sem checagem no código.

## Seguro vs. inseguro
- Inseguro: `Invoice.objects.get(id=pk)` — Seguro: `Invoice.objects.get(id=pk, owner=request.user)`
- Inseguro: `jwt.decode(token, options={"verify_signature": False})` — Seguro: `jwt.decode(token, key, algorithms=["HS256"])`
- Inseguro: `user.update(req.body)` — Seguro: copiar só os campos permitidos (DTO/serializer com allowlist).

## Perguntas de refutação
- Existe guarda global (middleware aplicado a todo o router, `DEFAULT_PERMISSION_CLASSES`, filtro do Spring Security)?
  Mostre onde e se cobre esta rota.
- O recurso é público por desenho (spec, contract)? Então não é achado.
- O ID é um UUID v4 e o recurso é só de leitura? Ainda é IDOR se o ID vaza em outro lugar; sem vazamento, exclusão nº 10.
- A checagem está no client-side apenas (exclusão nº 8) ou falta no servidor?

## Não reportar
Exclusões nº 8, 10, 11 e 22. Falta de MFA, política de senha ou lockout sem vulnerabilidade concreta.

## CWE comuns
CWE-862 (autorização ausente), CWE-863 (autorização incorreta), CWE-639 (IDOR), CWE-285, CWE-287 (autenticação),
CWE-347 (assinatura JWT), CWE-352 (CSRF), CWE-915 (mass assignment), CWE-384 (fixação de sessão).

## Correção típica
Usar o mecanismo de autorização que o projeto já tem, no mesmo nível das rotas vizinhas; filtrar por dono/tenant
na consulta; allowlist de campos graváveis; verificar assinatura e algoritmo do JWT.
