# Classe: supply-chain

Dependências com vulnerabilidade conhecida, dependências não fixadas, scripts de instalação perigosos, código
baixado em tempo de execução. Ferramentas: `tools.md` nº 10–16. Padrões de grep: `grep-patterns.md` § Supply chain.

## Quando a unidade existe no ledger
Uma unidade por manifest (e lockfile) de cada ecossistema, mais uma para scripts de build/instalação/CI.

## O que caçar
1. **Advisory conhecido** (vindo do SCA): pacote e versão afetados, versão corrigida, advisory (GHSA/CVE/OSV).
2. **Alcançabilidade:** a função vulnerável do pacote é usada pelo código de runtime? (govulncheck e alguns SCA
   informam; senão, Grep pelo import e pela API citada no advisory.) Alcançável → mantém severidade; não alcançável
   ou só em dev → rebaixa um nível.
3. **Sem lockfile versionado** → lacuna de reprodutibilidade (não vulnerabilidade).
4. **Versão solta** em dependência direta de runtime (`^`, `~`, `*`, `latest`, `>=`) sem lockfile → anote junto da lacuna.
5. **Scripts de instalação** do próprio projeto que baixam e executam código (`curl | sh`, `pip install` de URL,
   `npm` de tarball remoto, `go install ...@latest` em build).
6. **Dependência por URL/git sem hash ou commit fixo** (`git+https://...` sem `@<sha>`).
7. **Nome suspeito** (typosquatting óbvio de pacote popular) — só com evidência clara; nunca especule.

## Perguntas de refutação
- O advisory afeta a versão **resolvida** no lockfile (não só a faixa do manifest)?
- O pacote é só de desenvolvimento/teste?
- A condição do advisory se aplica (feature, plataforma, configuração usada pelo projeto)?

## Não reportar
Exclusões nº 14 (desatualizado sem advisory) e dependências que não existem no registry público por serem
internas (pacote privado não é typosquat).

## Severidade
Severidade do advisory (`tools.md` § 4), ajustada: dev-only ou não alcançável → um nível abaixo; sem versão
corrigida → "Precisa de verificação" com mitigação.

## CWE comuns
CWE-1395 (dependência vulnerável), CWE-1104 (componente sem manutenção), CWE-494 (download sem verificação de integridade), CWE-829.

## Correção típica
**Não aplicada pelo agente.** O achado traz o comando de atualização para o usuário rodar (ex.:
`npm install pacote@1.2.3`, `pip install "pacote==1.2.3"` + atualizar o requirements, `go get pacote@v1.2.3`),
a menor versão corrigida compatível e o aviso se for major (quebra provável → `toca contrato` se mudar saída
prometida). Tipo de commit: `build(deps)`.
