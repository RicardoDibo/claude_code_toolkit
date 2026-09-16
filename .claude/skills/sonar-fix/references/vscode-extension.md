# Referência — SonarQube for IDE (VS Code) como fonte de achados

## Pré-requisitos
- Extensão **SonarQube for IDE** (ID `SonarSource.sonarlint-vscode`) instalada e ativa.
- Claude Code rodando dentro do VS Code (extensão do Claude Code ou CLI conectado com `/ide`), para que a ferramenta de diagnósticos da IDE fique disponível.
- Opcional: **connected mode** configurado para o SonarQube Server/Cloud do time, para que as regras e o quality profile sejam os mesmos do pipeline.

## Formato dos diagnósticos
A ferramenta de diagnósticos devolve, por arquivo (URI), uma lista como esta:
```json
{
  "message": "Remove this useless assignment to variable \"x\".",
  "severity": "Warning",
  "range": { "start": { "line": 41, "character": 8 }, "end": { "line": 41, "character": 20 } },
  "source": "sonarqube",
  "code": "typescript:S1854"
}
```
- **Filtro:** `source` contendo `sonar`, sem diferenciar maiúsculas e minúsculas (versões antigas usam `sonarlint`).
- **Linha:** `range.start.line` é baseada em 0. Some 1 para citar `arquivo:linha`.
- **Regra:** `code` = `<repositório>:<regra>`. O número após `S` é o RSPEC.
- **Severidade:** é a severidade de exibição do VS Code (Error, Warning, Information, Hint), não a do Sonar. Para priorizar, use o tipo da regra (bug, vulnerabilidade, code smell) descrito na página da regra.

## Formato colado do painel Problems
Quando o usuário copia itens do painel Problems, cada item vem aproximadamente assim:
```
src/app/service.ts: Remove this useless assignment to variable "x". sonarqube(typescript:S1854) [Ln 42, Col 9]
```
Nesse formato a linha já começa em 1.

## Descrição das regras
- Web: `https://rules.sonarsource.com/<linguagem>/RSPEC-<número>/` (linguagem: `java`, `typescript`, `javascript`, `python`, `csharp`, `php`, `go`, `kotlin`, `html`, `css`, `docker`…).
- Na extensão: a ação rápida (lâmpada) do diagnóstico abre a descrição da regra. O usuário pode copiar o texto para o chat.

## Controle da análise (ações do usuário)
- A extensão analisa os arquivos ao abrir e ao salvar. Arquivos editados pelo Claude podem precisar ser abertos ou salvos para que os diagnósticos atualizem.
- Paleta de comandos (Ctrl+Shift+P) → digite `SonarQube` para ver os comandos da versão instalada, por exemplo a análise de arquivos alterados no VCS e a busca de hotspots numa pasta (connected mode).
- View **SonarQube** na barra lateral: status do connected mode, lista de hotspots e regras.

## Log da extensão
Painel **Output** → canal **SonarQube for IDE**. Para mais detalhes, ative `sonarlint.output.showAnalyzerLogs` / `sonarlint.output.showVerboseLogs`. Causas comuns de "nenhum achado aparece":
- runtime Node.js ou Java não encontrado (ver `sonarlint.pathToNodeExecutable` e `sonarlint.ls.javaHome`);
- arquivo fora da análise (exclusões de `sonarlint.analysisExcludesStandalone` ou do servidor em connected mode);
- linguagem não suportada no modo padrão (algumas só em connected mode);
- falha de autenticação ou sincronização no connected mode.

## Configurações que exigem aprovação para alterar
- `sonarlint.rules` (ativar ou desativar regras)
- `sonarlint.analysisExcludesStandalone`
- `sonarlint.connectedMode.*`
