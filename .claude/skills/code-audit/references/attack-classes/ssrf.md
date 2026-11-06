# Classe: ssrf

Server-Side Request Forgery: o servidor faz uma requisição de saída para um destino que o atacante escolhe.
Padrões de grep: `grep-patterns.md` § SSRF.

## Quando a unidade existe no ledger
Entry point cujo dado chega a um cliente HTTP/HTTPS, de webhook, de importação por URL, de preview de link,
de geração de PDF/imagem a partir de URL ou HTML, ou de conexão a host configurável pelo usuário.

## O que caçar
1. URL inteira vinda da requisição passada ao cliente HTTP (`requests.get(url)`, `fetch(req.body.url)`).
2. Host ou esquema montado com dado externo (`f"http://{host}/status"`).
3. Webhook/callback cadastrado pelo usuário e chamado pelo servidor sem restrição de destino.
4. Validação por string (`startswith("https://api.")`, regex no host) que redirect, `@`, IP decimal/hex,
   IPv6 ou DNS rebinding contornam.
5. Renderizadores (wkhtmltopdf, Puppeteer, LibreOffice, ImageMagick) que buscam recursos referenciados no
   conteúdo enviado pelo usuário.
6. Cliente que segue redirects automaticamente depois de validar só a URL inicial.

## Impacto que torna o achado real
Acesso a metadados de nuvem (`169.254.169.254`), a serviços internos sem autenticação, a portas administrativas,
ou leitura de arquivos via `file://`/`gopher://`. Sem destino interno alcançável demonstrável, rebaixe ou marque PLAUSIBLE
com o bloqueio "depende da topologia de rede em produção".

## Seguro vs. inseguro
- Inseguro: `requests.get(request.args["url"])` — Seguro: allowlist de hosts, resolução do DNS, bloqueio de faixas
  privadas/link-local no IP resolvido, sem seguir redirect (ou revalidando a cada salto).

## Perguntas de refutação
- O atacante controla host ou protocolo, ou só o path/query de um host fixo (exclusão nº 6)?
- A URL vem de configuração do servidor ou env (exclusão nº 9)?
- O código roda no client-side (browser)? SSRF em JS de cliente não é achado.
- A resposta volta ao atacante (SSRF completo) ou não (cego)? Isso muda a severidade, não a existência.

## Não reportar
Exclusões nº 6 e 9; SSRF em arquivos `.html` estáticos.

## CWE comuns
CWE-918 (SSRF), CWE-441 (proxy não intencional).

## Correção típica
Allowlist de destinos no padrão do projeto; validar o IP **resolvido**; desligar redirects; restringir esquemas a
`https`; isolar o egress na infraestrutura (recomendação, não código).
