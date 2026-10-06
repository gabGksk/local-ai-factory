# Configuração

Precedência: **padrões < `factory.toml` (em `--home`/`$FACTORY_HOME`/diretório atual) < variáveis de ambiente < flags da CLI**.
Gere o arquivo com `local-ai-factory init` (modelo em `factory.toml.example`).

```toml
[factory]
max_iterations = 50        # passos por execução (resume reinicia a contagem)
max_failures = 10          # falhas por execução
max_runtime_minutes = 240  # tempo total acumulado
max_task_retries = 5       # tentativas por tarefa / ciclos de debug antes de rollback
max_repairs = 2            # re-prompts para saída JSON inválida
test_command = "auto"      # auto (npm test | node --test | pytest -q) ou um comando literal, ex.: "go test ./..."

[llm]
provider = "ollama"        # ollama | mock
base_url = "http://localhost:11434"
model = "auto"
timeout_seconds = 300

[git]
enabled = true
auto_commit = true
checkpoints = true

[security]
sandbox = true
dangerous_commands = "blocked"
block_at = "HIGH"          # MEDIUM | HIGH | BLOCKED
command_timeout_seconds = 120
max_delete_files = 20

[agents]                   # false desativa reviewer/qa; desativar architect/tester/debugger bloqueia a execução com mensagem clara
manager = true
architect = true
developer = true
tester = true
debugger = true
reviewer = true
qa = true
```

## Variáveis de ambiente
`MAX_ITERATIONS`, `MAX_FAILURES`, `MAX_RUNTIME_MINUTES`, `MAX_TASK_RETRIES`, `TEST_COMMAND`, `LLM_PROVIDER`, `OLLAMA_BASE_URL`, `OLLAMA_MODEL`, `LLM_TIMEOUT_SECONDS`, `FACTORY_HOME`.
Valores inválidos são ignorados (vale o padrão). `MOCK_INJECT_BUG=0` desliga o bug proposital do provedor `mock`.
