# Template — Relatório de Debug

Salvar como: debug/BUG-YYYY-MM-DD-<slug>.md

```markdown
# BUG — [título curto do sintoma]

## Delta
Esperado: ... Observado: ... Condições: ...

## Evidências
### Imagens fornecidas
| # | Transcrição verbatim (erros, códigos, valores) | Estado da UI / metadados | Inferências (confirmadas?) |
|---|-----------------------------------------------|--------------------------|----------------------------|

### Outras evidências
- Logs / SELECTs (via db-reader) / trechos de código relevantes, com origem

## Funcionamento teórico
- Descrição do usuário (reformulada e confirmada)
- Artefatos SDD consultados: contract/spec de F0X — situação:
  [coincide | diverge da teoria do usuário | não cobre o caso]

## Reprodução
- Caso mínimo (comando/teste que falha): ...
- Frequência: sempre | intermitente (padrão: ...)

## Hipóteses testadas
| # | Hipótese | Teste executado | Resultado (confirma/elimina) |
|---|----------|-----------------|------------------------------|

## Diagnóstico
- Causa raiz: ...
- Cadeia de evidências: A + B + C → conclusão
- Classificação: código | spec/contrato | dado | integração
- Por que não foi pego antes: [CA faltante | caso de borda ausente no
  spec | checkpoint fraco | sem guarda de regressão]

## Correção
- O que mudou (arquivos/artefatos/dados) e por quê ataca a causa raiz
- Teste de regressão adicionado: [caminho do teste — obrigatório]
- Rota SDD: [feature-implementer | sdd-feature-author | script de dado]
- Verificação pós-correção: [checkpoint/cenário re-executado, evidência]
- Commit: [hash — via /commit, tipo fix]

## Prevenção
- Artefato atualizado (CA/caso de borda/linha na Matriz): ...
- Sintomas para busca futura (palavras-chave): ...
```
