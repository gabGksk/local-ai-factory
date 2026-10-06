# Quickstart

```bash
local-ai-factory init                                  # cria factory.toml e projects/
local-ai-factory doctor                                # diagnóstico
local-ai-factory create meu-app --spec "Sistema de estoque: produtos, fornecedores, entradas/saídas, testes."
local-ai-factory run meu-app --autonomous              # roda até concluir ou parar com segurança
```

Saída final: `PROJECT COMPLETED` ou
```
PROJECT STOPPED SAFELY
Reason: Maximum iterations reached
Last stable checkpoint: tests-passing
Pending: 3 tasks
```

## Comandos
| Comando | Função |
|---|---|
| `init` / `doctor` | configuração inicial / diagnóstico do ambiente |
| `create NOME --spec TEXTO \| --spec-file ARQ` | cria projeto (workspace isolado + Git) |
| `run [NOME] [--autonomous] [--provider mock] [--model X]` | executa o loop (sem `--autonomous` pergunta entre passos) |
| `status [NOME]` | painel (agentes, iteração, testes, tarefa atual) |
| `logs [NOME] --agent developer -n 50` | logs por agente |
| `test [NOME]` | roda a suíte do projeto |
| `stop [NOME]` | pede parada segura (outro terminal) |
| `resume [NOME]` | continua de onde parou (inclusive após queda de energia) |
| `rollback [NOME] [--to CP] [--list]` | volta a um checkpoint (padrão: último estável) |
| `projects` | lista projetos |

## Modo autônomo / noturno
`run --autonomous` não pede nada, registra tudo, cria checkpoints, recupera falhas e respeita
`MAX_ITERATIONS`, `MAX_FAILURES`, `MAX_RUNTIME_MINUTES`, `MAX_TASK_RETRIES`. Ao bater um limite encerra em estado seguro e gera `reports/`.
Exemplo noturno: `MAX_RUNTIME_MINUTES=480 local-ai-factory run meu-app --autonomous`.

## Recuperação e rollback
* Interrompeu (Ctrl+C, queda de energia)? `local-ai-factory resume meu-app` — o estado é gravado atomicamente a cada passo.
* Algo quebrou? `local-ai-factory rollback meu-app --list` e `rollback meu-app --to tests-passing`, depois `resume`.
* Quando uma correção falha `MAX_TASK_RETRIES` vezes, a Factory faz rollback sozinha ao último checkpoint estável e para com diagnóstico.

## Onde está tudo (por projeto)
```
projects/meu-app/  src/ tests/ docs/ architecture/ memory/ logs/ reports/ .factory/state.json  (+ .git)
```
