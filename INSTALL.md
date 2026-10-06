# Instalação

## Requisitos
* Python **3.11+** (usa `tomllib`)
* Git (recomendado; sem ele os checkpoints usam cópias de arquivos)
* [Ollama](https://ollama.com) + pelo menos um modelo, para uso real (opcional para o modo `mock`)

## Passo a passo
**Linux/macOS**
```bash
git clone <seu-repo> local-ai-factory && cd local-ai-factory
./scripts/install.sh
. .venv/bin/activate
```
**Windows (PowerShell)**
```powershell
git clone <seu-repo> local-ai-factory; cd local-ai-factory
powershell -ExecutionPolicy Bypass -File scripts\install.ps1
.\.venv\Scripts\Activate.ps1
```
**Manual (qualquer SO)**
```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .\.venv\Scripts\Activate.ps1
pip install -e ".[dev]"
local-ai-factory init
```

## Ollama
```bash
ollama serve                      # normalmente já roda como serviço
ollama pull qwen2.5-coder:7b      # sugestão para máquinas com ~8 GB de RAM; modelos maiores = melhores resultados
local-ai-factory doctor           # confirma servidor e modelos
```
Com `model = "auto"` a Factory escolhe o melhor modelo instalado (prefere *coder*, ignora modelos de embedding).
Para fixar um: `OLLAMA_MODEL=llama3:8b` ou `model = "llama3:8b"` no `factory.toml`.

## Verificação
```bash
local-ai-factory doctor    # deve terminar com READY
pytest                     # suíte completa
```

## Solução de problemas
* `error in 'egg_base' option: 'src' does not exist`: o clone está incompleto (faltam `src/`, `tests/`, `scripts/`). Confira com `git ls-files src | Select-Object -First 3`; se vazio, o código não foi commitado/enviado ao repositório — faça `git add -A; git commit; git push` a partir da pasta completa.
* O workflow `.github/workflows/ci.yml` roda a instalação, `pytest` e o fluxo `mock` em Windows e Linux a cada push.
