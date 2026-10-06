# Arquitetura

## Visão geral
```
CLI (cli.py) ──► Orchestrator/Manager (orchestrator.py) ──► AgentRunner (agents.py) ──► LLMProvider (llm/)
                        │                                        │ ações validadas
                        ▼                                        ▼
        StateManager · TaskBoard · Reports          Safety Engine ► CommandExecutor / FileSystemEngine / GitEngine
                        │
                 ProjectMemory · ContextManager · FactoryLogger
```
O core não conhece a CLI: uma Web UI ou dashboard pode reutilizar `Project` + `Orchestrator` sem reescrever nada.

## Módulos (`src/laf/`)
| Módulo | Responsabilidade |
|---|---|
| `config.py` | defaults < `factory.toml` < variáveis de ambiente; validação |
| `safety.py` | classifica comandos (SAFE…BLOCKED), confina caminhos, rejeita operadores de shell |
| `executor.py` | `CommandExecutor`: subprocess real, stdout/stderr/exit code/duração, timeout, cancelamento, perfis de terminal |
| `fs.py` | `FileSystemEngine`: tudo confinado ao `PROJECT_ROOT` (bloqueia `..`, symlink, `.git`, `.factory`, deleção em massa) |
| `git_engine.py` | init/status/diff/add/commit/branch/checkout/log/tag/restore + `checkpoint`/`rollback` (fallback sem Git: snapshots) |
| `state.py` | `Task`/`TaskBoard` (dependências, DAG) e `ProjectState` persistido atomicamente |
| `memory.py` | memória em markdown (decisões, erros, soluções, descobertas, lições) + recall por palavras-chave |
| `context.py` | `ContextManager`: prompts limitados; só memória/arquivos relevantes; dados não confiáveis delimitados |
| `llm/` | `LLMProvider` (ABC), `OllamaProvider` (HTTP, auto-modelo, timeout), `MockProvider`, `ScriptedProvider` |
| `structured.py` | extrai JSON, valida por papel, **reparo por retry** com os erros exatos |
| `agents.py` | `AGENT_SPECS` (permissões explícitas) e `AgentRunner` (executa ações propostas pelo LLM) |
| `testrunner.py` | resolve o comando de teste (`auto`/literal) e interpreta resultados de pytest, unittest, node, jest |
| `orchestrator.py` | máquina de estados, limites, verificação por evidência, rollback, resume |
| `reports.py` / `doctor.py` / `cli.py` | relatórios, diagnóstico, interface |

## Máquina de estados
`INIT → ANALYZE → PLAN → IMPLEMENT → TEST → REVIEW → QA → ACCEPTANCE → RELEASE`
Retornos: `TEST→DEBUG→TEST`, `REVIEW→IMPLEMENT`, `QA→DEBUG`, `ACCEPTANCE→DEBUG`.
Checkpoints: `init`, `architecture-complete`, `implementation-vN`, `pre-debug-N`, `tests-passing`, `qa-approved`, `stable-release`.

## Protocolo agente ↔ sistema
O LLM devolve **um objeto JSON**:
```json
{"status":"success","summary":"...","actions":[{"type":"write_file","path":"src/x.py","content":"..."},{"type":"run","command":"pytest -q"}]}
```
O sistema valida o schema → autoriza cada ação (permissões do agente + Safety Engine) → executa → **verifica evidências**.
O LLM nunca executa nada diretamente.

## Como estender
**Novo agente:** adicione um `AgentSpec` em `AGENT_SPECS` (nome, `Permissions(fs, terminal, write_prefixes)`, instruções) e
um passo no `Orchestrator` que chame `self.runner.run("meu-agente", phase, ...)`. Adicione o papel em `validate_output` se precisar de campos próprios.

**Nova ferramenta/ação:** inclua o tipo em `ACTION_TYPES` (`structured.py`), valide o payload em `validate_output`
e trate-o em `AgentRunner._execute` — passando sempre por `SafetyEngine`/`FileSystemEngine`.

**Novo provedor de LLM:** implemente `LLMProvider` (`available`, `list_models`, `generate`) em `llm/` e registre em `get_provider`.

**Novo modelo Ollama:** `ollama pull <modelo>` e `OLLAMA_MODEL=<modelo>` (ou `auto`).

**Nova regra de segurança:** acrescente uma tupla `(regex, nível, motivo)` em `safety.RULES` e um teste em `tests/safety/`.

## Decisões
* **Python stdlib, sem dependências em runtime:** instalação trivial, estabilidade, ótimo suporte a CLI/testes (pytest só em dev).
* **JSON em vez de function-calling nativo:** funciona com qualquer modelo local, inclusive os que não têm *tool use*.
* **Sem concorrência na V1:** prioridade correção > observabilidade > segurança > determinismo > performance. A fronteira `AgentRunner` permite paralelizar depois.
* **Estado em JSON atômico (tmp + `os.replace`):** sobrevive a queda de energia sem banco de dados.
