# LOCAL AI FACTORY

Fábrica de software **autônoma, local e gratuita**: você descreve um projeto, uma equipe de agentes de IA
(Manager, Architect, Developer, Tester, Debugger, Reviewer, QA) projeta, implementa, testa, depura, revisa e valida
— tudo em um workspace isolado, com Git, memória persistente, logs e relatórios.

```
ESPECIFICAÇÃO → MANAGER → ANALYZE → PLAN → IMPLEMENT → TEST ⇄ DEBUG → REVIEW → QA → ACCEPTANCE → RELEASE
```

* **Local-first / gratuito:** Python (stdlib, **zero dependências em runtime**) + [Ollama](https://ollama.com). Sem APIs pagas.
* **Evidência, não promessa:** uma tarefa só é `COMPLETED` quando a Factory verifica (arquivos existem, código compila,
  testes **executados por ela** passam, critérios de aceitação rodam de verdade).
* **Seguro por construção:** Safety Engine em toda execução, permissões por agente, workspace confinado, dados do projeto tratados como não confiáveis.
* **Recuperável:** estado atômico em disco (`resume`), checkpoints Git, `rollback`.

## Uso rápido

```bash
./scripts/install.sh            # Windows: powershell -ExecutionPolicy Bypass -File scripts\install.ps1
. .venv/bin/activate            # Windows: .\.venv\Scripts\Activate.ps1
local-ai-factory doctor
local-ai-factory create estoque --spec-file minha-spec.md
local-ai-factory run estoque --autonomous
local-ai-factory status estoque
```

Sem Ollama? Teste toda a infraestrutura offline: `local-ai-factory run estoque --autonomous --provider mock`
(cenário determinístico "Todo list" com um bug proposital que o Debugger corrige).

## Documentação

| Arquivo | Conteúdo |
|---|---|
| [INSTALL.md](INSTALL.md) | Instalação e requisitos |
| [QUICKSTART.md](QUICKSTART.md) | Primeiros 10 minutos, modo autônomo, resume, rollback |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Componentes, fluxo, como criar agentes/ferramentas/provedores |
| [SECURITY.md](SECURITY.md) | Safety Engine, permissões, sandbox, injeção de prompt |
| [CONFIGURATION.md](CONFIGURATION.md) | `factory.toml`, variáveis de ambiente, limites |
| [docs/MODELS.md](docs/MODELS.md) | Adicionar modelos Ollama e criar provedores de LLM |
| [docs/AGENTS-AND-TOOLS.md](docs/AGENTS-AND-TOOLS.md) | Criar agentes/ferramentas; comando de teste e outras stacks (Node etc.) |
| [docs/OPERATIONS.md](docs/OPERATIONS.md) | Modo autônomo, `resume` e `rollback` |
| [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) | Desenvolver a própria Factory |
| [STATUS.md](STATUS.md) | **O que foi verificado e o que NÃO foi** (leia antes de usar com modelo real) |

## Desenvolvimento

```bash
pip install -e ".[dev]" && pytest          # 170+ testes: unit, integração, e2e, safety, agents, orchestrator
./scripts/dogfood.sh                       # a Factory constrói um projeto sozinha
```
