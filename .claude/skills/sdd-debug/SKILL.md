---
name: sdd-debug
description: Conduz uma sessão de debug estruturada a partir de evidências fornecidas pelo usuário — screenshots/prints (telas de erro, UI quebrada, console, logs fotografados), descrição do funcionamento teórico esperado, stack traces ou mensagens de erro. Use esta skill sempre que o usuário reportar um bug, comportamento inesperado, erro, tela quebrada, "não está funcionando", divergência entre o esperado e o observado, ou enviar prints de erro — em projetos SDD ou não.
---

# SDD Debug — Debug orientado a evidências

Você é um engenheiro de debug metódico. O objetivo NUNCA é "fazer o erro sumir" — é encontrar a causa raiz com uma cadeia de evidências e corrigir com proteção contra regressão. Regra de ouro: **nenhuma correção antes de reproduzir o problema ou reunir evidência forte da causa**.

## Fase 0 — Coleta de evidências

### Imagens (protocolo obrigatório quando o usuário envia prints)
Imagens são evidência primária, mas evidência ≠ conclusão. Para cada imagem:
1. **Transcreva verbatim** todo texto relevante: mensagens de erro completas, códigos de status, stack traces visíveis, URLs, nomes de campos/botões. A transcrição vai para o relatório — nunca parafraseie uma mensagem de erro.
2. **Registre o estado da UI**: o que está renderizado, o que está faltando, valores exibidos, estados de loading/disabled.
3. **Capture metadados visíveis**: timestamps, versão do app, ambiente (URL de staging vs prod), navegador/dispositivo, usuário logado.
4. **Separe explicitamente** "visível na imagem" de "inferido por mim". Inferências precisam ser confirmadas em código/log antes de virar diagnóstico.
5. Se a imagem estiver ilegível ou cortada no ponto crítico, peça outra captura específica ("consegue capturar a aba Network com a request que falha aberta?") em vez de adivinhar.

### Funcionamento teórico (descrição do usuário)
6. Reformule o comportamento esperado em passos concretos e confirme com o usuário.
7. **Projeto SDD:** cruze com os artefatos — leia o `contract.md` e o `spec.md` da feature envolvida (via `manifest.md`/`features/`). Três situações, registre qual é:
   - Teoria do usuário == artefatos → esperado formalizado; bug provável no código.
   - Teoria ≠ artefatos → conflito de expectativa: o "bug" pode ser o sistema cumprindo o contrato. Mostre a divergência ANTES de investigar código — talvez o que precisa mudar seja o artefato (mudança de escopo, não bug).
   - Não coberto pelos artefatos → gap de spec; o debug vai gerar atualização de artefato além da correção.

### Contexto restante
8. Colete apenas o bloqueante: passos para reproduzir, frequência, quando começou (o que mudou — deploy, migração, dado novo?), logs se acessíveis.

**Gate:** você consegue enunciar o delta (Fase 1) sem nenhum "acho que". Senão, volte e pergunte.

## Fase 1 — Delta
Uma frase precisa: **"Esperado: X. Observado: Y. Condições: Z."** O delta é o contrato da sessão de debug — tudo o que vier depois serve para explicá-lo.

## Fase 2 — Reprodução
Caso mínimo (curl direto na rota antes de UI inteira; teste unitário antes de e2e). Reproduziu → congele como script/teste que falha (vira o teste de regressão da correção). Não reproduziu → delta incompleto; volte à Fase 0 com perguntas dirigidas. Intermitente: procure o padrão (concorrência? cache? dado específico?) antes de prosseguir.

## Fase 3 — Hipóteses ranqueadas
Máximo 5, ordenadas por probabilidade × custo de verificar. Para CADA uma, defina o teste barato que a confirma ou elimina ANTES de executá-lo. UMA variável por vez.

Táticas por evidência:
- Stack trace no print → vá à linha; leia o código real (Read/Grep), não a memória dele.
- Erro 4xx/5xx → bisecte a fronteira: a request chega? o handler executa? a query roda?
- "Funcionava antes" → bisecte o tempo: `git log` do módulo desde a última versão boa.
- UI errada com API correta → estado do frontend (o dado chega e é mal renderizado, ou nem chega?).
- **Hipótese de dado** → delegue ao agente `db-reader` os SELECTs de verificação; nunca "corrija" olhando só o código se a causa pode estar no dado.

## Fase 4 — Diagnóstico
Causa raiz com cadeia de evidências explícita: A (print transcrito) + B (log/SELECT) + C (linha de código) → conclusão. Elo "provavelmente" = diagnóstico não pronto. Pergunte também **por que não foi pego antes** (CA faltante? caso de borda ausente? checkpoint fraco?).

Classifique: **código** (artefato certo, implementação diverge) | **spec/contrato** (implementação cumpre artefato errado/ambíguo) | **dado** (estado inválido — descubra COMO chegou lá, esse é o bug real) | **integração** (contratos individuais ok, fronteira mal definida — gap na Matriz).

## Fase 5 — Correção roteada
- **Código** → correção mínima na causa raiz + teste da Fase 2 promovido a regressão. Delegue ao `feature-implementer`: aplicar SOMENTE esta correção, re-rodar checkpoints, sem tocar saídas do contract.md.
- **Spec/contrato** → skill `sdd-feature-author` atualiza artefato + CA novo ANTES de tocar código; breaking em saída prometida exige aprovação do usuário primeiro.
- **Dado** → script de correção reversível (nunca UPDATE ad-hoc sem registro) + correção da validação que permitiu o estado; `db-reader` valida antes/depois.
- **Integração** → linha na Matriz do PRD + contratos dos dois lados + cenário de integração novo, via `sdd-feature-author`.

Correção verificada = commit via fluxo `/commit` (tipo `fix`, corpo referenciando `debug/BUG-...md`). Fora de projeto SDD: mesmos princípios sem roteamento — correção mínima + regressão, sempre.

## Fase 6 — Relatório
Use `references/bug-report-template.md` (leia antes). Salve em `debug/BUG-<data>-<slug>.md`. Antes de cada sessão nova, Grep em `debug/` por sintomas parecidos — bugs recorrentes são reconhecidos, não re-investigados.

## Regras da sessão
- Nunca proponha "limpar cache / reinstalar / try-catch" como primeira resposta — supressão de sintoma não é debug.
- Erro transcrito > erro lembrado. Código lido > código presumido. Dado consultado > dado imaginado.
- 3 rodadas de hipóteses sem confirmação → pare e re-examine o delta; a premissa costuma estar errada.
- Bug corrigido sem teste de regressão não está corrigido; está adiado.
