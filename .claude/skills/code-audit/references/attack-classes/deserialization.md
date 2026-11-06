# Classe: deserialization

Desserialização insegura e execução dinâmica de código: pickle, YAML inseguro, Java/.NET serialization,
PHP `unserialize`, `eval`/`exec`/`new Function`/`vm`. Padrões de grep: `grep-patterns.md` § Desserialização.

## Quando a unidade existe no ledger
Algum dado que atravessa uma fronteira de confiança (requisição, fila, cache compartilhado, arquivo enviado,
cookie, banco escrito por outro sistema) chega a um desserializador ou avaliador de código.

## O que caçar
1. `pickle.loads`/`joblib.load`/`dill` sobre bytes vindos de fora (inclui cache Redis compartilhado e modelos de ML baixados).
2. `yaml.load` sem `SafeLoader` (ou `yaml.unsafe_load`, `Loader=yaml.Loader`/`FullLoader` em versões antigas).
3. Java `ObjectInputStream.readObject`, XStream, Jackson com default typing polimórfico; .NET `BinaryFormatter`,
   `TypeNameHandling` diferente de `None` no Json.NET.
4. PHP `unserialize` em entrada do usuário; Ruby `Marshal.load`, `YAML.load` (Psych < 4).
5. `eval`, `exec`, `new Function`, `vm.runInContext`, `setTimeout(string)`, `ScriptEngine.eval` com dado externo.
6. Cookie ou token assinado com segredo fraco/hardcoded que depois é desserializado (ex.: sessão em pickle) —
   combine com `secrets`.

## Seguro vs. inseguro
- Inseguro: `data = pickle.loads(request.data)` — Seguro: `json.loads(request.data)` com validação de schema.
- Inseguro: `yaml.load(body)` — Seguro: `yaml.safe_load(body)`.
- Inseguro: `eval(req.query.expr)` — Seguro: parser de expressão restrito ou tabela de operações permitidas.

## Perguntas de refutação
- A origem dos bytes é realmente controlável por quem está fora da fronteira? Arquivo gerado e lido pelo próprio
  processo, em diretório não gravável por terceiros, não é achado.
- Há assinatura/HMAC verificada **antes** de desserializar, com chave forte e fora do código?
- O `eval` recebe só constantes ou valores de allowlist?

## Não reportar
Exclusões nº 9 e 15. Desserialização de JSON comum (não polimórfico) sem tipo dinâmico.

## CWE comuns
CWE-502 (desserialização), CWE-95 (eval injection), CWE-94 (injeção de código).

## Correção típica
Trocar o formato por JSON/Protobuf com schema; usar o loader seguro; desligar typing polimórfico; remover o
`eval` por tabela de despacho. Mudança de formato persistido ou trafegado → `toca contrato: sim`.
