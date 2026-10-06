# STATUS — verificado vs. não verificado

**Veredito: PARTIALLY COMPLETE** (infraestrutura 100% verificada; execução com modelo Ollama real **não** foi possível aqui).

Esta Factory foi construída e testada num sandbox Linux (Python 3.12, Git 2.43) **sem Ollama e sem acesso à sua máquina**.
Tudo abaixo foi executado de verdade, exceto o que está em "Não verificado".

## Critérios de aceitação (`tests/e2e/test_acceptance.py`)
| AC | Critério | Evidência | Estado |
|---|---|---|---|
| 01 | Inicialização | `python -m laf --version`, CLI, install.sh em venv limpo | ✅ |
| 02 | Ollama detecta servidor/modelos | `OllamaProvider` contra servidor HTTP real que fala a API do Ollama (`/api/tags`, `/api/chat`) | ⚠️ protocolo ✅ · daemon real ❌ |
| 03 | Criar projeto | `create` + workspace + Git | ✅ |
| 04 | 7 agentes | `AGENT_SPECS` com permissões | ✅ |
| 05 | Fluxo multiagente | todos os agentes aparecem nos logs de uma execução | ✅ |
| 06 | Criar/modificar arquivos | confinados ao workspace | ✅ |
| 07 | Terminal autorizado | executor real + perfis + Safety | ✅ |
| 08 | Testes reais | pytest executado pela Factory | ✅ |
| 09 | Debug | falha detectada → Debugger → correção → re-teste | ✅ |
| 10 | Checkpoints Git | tags `checkpoint/*` | ✅ |
| 11 | Rollback | `git reset --hard` ao checkpoint (+ fallback sem Git) | ✅ |
| 12 | Memória | decisões/erros/soluções persistem e são recuperadas | ✅ |
| 13 | Safety Engine | 30+ comandos perigosos recusados | ✅ |
| 14 | Autonomia | 13 iterações sem intervenção; limites; resume após crash | ✅ |
| 15 | Relatórios | `final-report.md` + 4 relatórios | ✅ |
| 16 | Projeto de demonstração | `demo/todo-demo` (bug proposital corrigido automaticamente) | ✅ (ver ressalva do mock) |

**173 testes passam** (unit, integração, e2e, safety, agents, orchestrator), inclusive na cópia instalada via `install.sh`.
> ⚠️ Na última sessão de trabalho o sandbox não tinha rede nem `pytest`; a suíte foi executada com um *runner mínimo compatível com pytest* (fora do projeto, só para verificação). **Rode `pip install -e ".[dev]" && pytest` na sua máquina** para confirmar com o pytest real — se algo divergir, é o runner de verificação, não o projeto, o primeiro suspeito, mas confirme.

## NÃO verificado — leia antes de usar
1. **Nenhum LLM real dirigiu o loop.** O dogfooding usa o `MockProvider`, que é *roteirizado*: ele produz o bug e a correção.
   Isso prova que toda a infraestrutura (arquivos, terminal, pytest, Git, estado, Safety, verificação por evidência) funciona
   e reage a uma falha real — **não** prova que um modelo 7B consegue escrever o projeto. Essa prova só existe na sua máquina:
   `local-ai-factory run meu-projeto --autonomous` com Ollama.
2. **Qualidade de modelos pequenos.** O ponto mais frágil é a saída JSON válida e o código completo em `write_file`.
   Há validação + reparo (re-prompt) e verificação por evidência, mas modelos fracos podem esgotar `max_task_retries`; nesse caso a
   Factory faz rollback e para com diagnóstico (comportamento testado). Prefira modelos *coder* de 7B+ ou maiores.
3. **Windows:** `install.ps1`, a detecção de RAM via `wmic` e o tratamento de caminhos `C:\` no Safety Engine nunca rodaram no Windows
   (só Linux). O código usa `sys.executable`, `shlex`, `pathlib` e evita shell para ser portável, mas **espere ajustes**.
4. **Projetos não-Python:** `factory.test_command = "auto"` detecta `npm test` / `node --test` / `pytest -q` e aceita qualquer comando literal (`TEST_COMMAND`). Verificado com um projeto Node real (`node --test`, passa e falha de verdade). Outras stacks (go, cargo, …) via `test_command` **não foram executadas aqui**; o parser entende pytest, unittest, node e jest/vitest — outros formatos caem em "nenhum teste executado" até você estendê-lo em `testrunner.py`.
5. **Segurança** é política por regras, não sandbox de SO (ver `SECURITY.md`).
6. **Sem concorrência** (decisão da V1).
7. Para projetos reais, `max_iterations=50` pode ser curto; ajuste os limites.

## Defeitos reais que os testes pegaram durante a construção (e foram corrigidos)
* Tester sem permissão de escrita em `tests/` (erro na minha tabela de permissões; o orquestrador bloqueou e fez rollback como projetado).
* Safety Engine deixava passar caminhos Windows (`C:\Windows\...`) porque `shlex` consome barras invertidas.
* `doctor` omitia a linha de modelos quando o Ollama estava offline.
* Logs/memória contavam como "arquivos alterados" pelo Debugger (poderia validar um fix falso); agora são excluídos do diff.

## Próximo passo recomendado (na sua máquina)
```bash
./scripts/install.sh && . .venv/bin/activate
local-ai-factory doctor                       # confirma Ollama + modelos reais
./scripts/dogfood.sh                          # baseline offline (mock)
LLM_PROVIDER=ollama ./scripts/dogfood.sh      # a mesma tarefa, agora com seu modelo
```
Se o último falhar, `local-ai-factory logs todo-demo --agent developer` e `reports/` dizem exatamente onde.
