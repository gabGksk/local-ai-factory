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
