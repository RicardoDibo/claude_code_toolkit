# Classe: iac

Infraestrutura como código, containers e CI/CD: Dockerfile, docker-compose, Kubernetes, Terraform/CloudFormation,
GitHub Actions/GitLab CI. Ferramentas: `tools.md` nº 17–19. Padrões de grep: `grep-patterns.md` § IaC.

## Quando a unidade existe no ledger
Uma unidade por arquivo de infraestrutura ou workflow de CI versionado no repositório.

## O que caçar
1. **Container:** roda como root sem necessidade; imagem `:latest` sem digest; `ADD` de URL remota; segredo em
   `ENV`/`ARG` ou copiado para a imagem; porta de debug exposta.
2. **Orquestração:** `privileged: true`, `hostNetwork`, `hostPath` sensível, `allowPrivilegeEscalation`, capabilities extras,
   sem limites quando o spec exige isolamento.
3. **Nuvem:** bucket/storage público, security group `0.0.0.0/0` em porta administrativa ou de banco, banco
   `publicly_accessible`, criptografia em repouso desligada quando o spec exige, IAM com `*` em ação e recurso.
4. **CI/CD:** `pull_request_target` que faz checkout do código do fork e executa; expressão `${{ github.event.* }}`
   de título/corpo/branch interpolada em `run:` (injeção de comando); segredos expostos a workflows de fork;
   action de terceiro sem versão fixada por SHA em workflow com segredos; `permissions` amplas por padrão.
5. **Configuração da aplicação versionada:** `DEBUG=True`, CORS `*` com credenciais, cookies inseguros em
   arquivo de produção (combine com `authz`).

## Perguntas de refutação
- O arquivo é de produção ou de desenvolvimento local (`docker-compose.dev.yml`, `.devcontainer`)? Local → rebaixa ou descarta.
- A entrada interpolada no workflow é controlável por quem não tem acesso de escrita ao repo?
- O recurso público é público por desenho (site estático, assets)?

## Não reportar
Exclusões nº 11 (hardening genérico) e regras de "boa prática" sem exposição concreta (ex.: falta de label,
de healthcheck, de `readOnlyRootFilesystem` sem caminho de ataque).

## CWE comuns
CWE-250 (privilégio desnecessário), CWE-732 (permissão incorreta), CWE-284 (controle de acesso), CWE-78 (injeção em `run:`),
CWE-1357 (componente não fixado).

## Correção típica
Usuário não root; imagem com digest; menor privilégio em IAM/K8s; mover a expressão de `${{ }}` para variável de
ambiente e referenciá-la entre aspas no script; fixar actions por SHA; restringir `permissions`. Mudanças que
afetam deploy → anote no relatório que exigem validação do usuário em ambiente não produtivo.
