# Segurança

## Camadas
1. **Safety Engine** (`safety.py`) — todo comando passa por ele *antes* de executar; o `CommandExecutor` não tem caminho que o ignore.
   Níveis: `SAFE · LOW_RISK · MEDIUM_RISK · HIGH_RISK · BLOCKED`. Política padrão: bloqueia `HIGH_RISK` e acima (`security.block_at`).
   Detecta: `rm -rf`, formatação/partição de disco, shutdown/reboot, acesso a chaves e credenciais (`.ssh`, `.aws`, navegadores), push forçado,
   `git reset --hard`/`clean`, `curl | sh`, `sudo`, `kill`, caminhos fora do projeto (inclui `C:\...` e `..`), operadores de shell (`; & | > < \` $()`).
2. **Execução sem shell** — comandos viram `argv` (`shlex`), nunca `shell=True`; interpretadores são mapeados para o Python atual.
3. **Permissões por agente** (`agents.AGENT_SPECS`):

| Agente | Escrita | Terminal |
|---|---|---|
| architect | `architecture/`, `docs/` | nenhum |
| developer | `src/`, `docs/`, `scripts/`, README, requirements | restrito (python, pytest, node) |
| tester | `tests/` | apenas pytest/unittest |
| debugger | `src/`, `docs/` (não pode alterar testes) | apenas pytest/unittest |
| reviewer | nenhuma | nenhum |
| qa | nenhuma | apenas pytest/unittest |
| manager | interno (orquestração) | completo, **ainda passa pelo Safety Engine** |

4. **Workspace confinado** (`fs.py`) — caminhos resolvidos com `realpath`; `..`, symlinks para fora, `.git/` e `.factory/` são recusados;
   deleção em massa (> `max_delete_files`) bloqueada; a raiz do projeto nunca é apagada.
5. **Injeção de prompt** (`context.py`) — conteúdo de arquivos, memória e saída de ferramentas entra em blocos `<untrusted>`;
   tags forjadas são neutralizadas; o system prompt manda tratar esse conteúdo apenas como dado. Mesmo que um modelo seja enganado,
   as camadas 1–4 impedem a ação (testado em `tests/agents`).
6. **Auditoria** — bloqueios vão para `logs/safety.log`.

## Limites honestos
* O Safety Engine é uma **política baseada em regras**, não um sandbox de SO. Código que o próprio projeto gera e que é executado
  (por exemplo, os testes) roda com os privilégios do seu usuário. Para código não confiável, rode a Factory dentro de uma VM ou
  contêiner Docker ou de um usuário sem privilégios.
* Regras por regex podem ter falsos negativos para comandos exóticos; amplie `RULES` e `block_at` conforme seu risco.
* `security.sandbox = false` desliga o bloqueio (os níveis continuam sendo classificados/logados). Não recomendado.
