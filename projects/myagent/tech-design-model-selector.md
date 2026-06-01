# Technical Design: Model Selector (#3)

**Author**: Hisham (Tech Lead)
**Created**: 2026-05-21
**PRD**: [prd-model-selector.md](./prd-model-selector.md)
**AgDRs**: AgDR-0003 (modularisation), AgDR-0004 (selection UX)

---

## Architecture

### New file: `src/models.py`

Owns all model-selection logic. `main.py` calls `select_model()` once before the chat loop.

```
src/
├── main.py        ← updated: imports select_model(), calls it at startup
└── models.py      ← new: fetch, display, select, fallback
```

---

## `src/models.py` — full design

### Constants

```python
OLLAMA_TAGS_URL = "http://localhost:11434/api/tags"
DEFAULT_MODEL = "smollm2:135m"
FETCH_TIMEOUT = 3  # seconds

# Static metadata for known models (name prefix → metadata)
MODEL_METADATA: dict[str, dict] = {
    "smollm2":      {"context": "8k",   "best_for": "Quick tasks"},
    "llama3.2":     {"context": "128k", "best_for": "General chat"},
    "llama3.1":     {"context": "128k", "best_for": "General chat"},
    "qwen2.5":      {"context": "32k",  "best_for": "General purpose"},
    "qwen2.5-coder":{"context": "32k",  "best_for": "Coding"},
    "mistral":      {"context": "32k",  "best_for": "Instruction following"},
    "deepseek-r1":  {"context": "64k",  "best_for": "Reasoning"},
    "phi4":         {"context": "16k",  "best_for": "Compact reasoning"},
    "gemma3":       {"context": "128k", "best_for": "General chat"},
    "nomic-embed":  {"context": "8k",   "best_for": "Embeddings"},
}
```

### Data model

```python
from dataclasses import dataclass

@dataclass
class Model:
    name: str          # e.g. "smollm2:135m"
    type: str          # "Local" | "Cloud"
    context: str       # e.g. "8k" — from metadata or "—"
    best_for: str      # e.g. "Quick tasks" — from metadata or "—"
    available: bool    # True = green, False = orange
    unavail_reason: str = ""  # shown in Status column when not available
```

### Functions

#### `list_local_models() -> list[Model]`

```python
def list_local_models() -> list[Model]:
    resp = requests.get(OLLAMA_TAGS_URL, timeout=FETCH_TIMEOUT)
    resp.raise_for_status()
    tags = resp.json().get("models", [])
    models = []
    for tag in tags:
        name = tag["name"]
        meta = _lookup_meta(name)
        models.append(Model(
            name=name, type="Local",
            context=meta["context"], best_for=meta["best_for"],
            available=True,
        ))
    return models
```

#### `check_subscription() -> list[Model]`

Cloud subscription API is undocumented. Returns empty list (graceful stub). When API is confirmed, this function is the sole change point.

```python
def check_subscription() -> list[Model]:
    # Stub: Ollama cloud subscription API endpoint not yet documented.
    # Returns empty list — Cloud group shows "No active subscription".
    return []
```

#### `_lookup_meta(name: str) -> dict`

Matches the model name prefix against `MODEL_METADATA`. Falls back to `{"context": "—", "best_for": "—"}`.

#### `select_model(console: Console) -> str`

Main entry point called by `main.py`. Returns the selected model name string.

```
1. list_local_models() — on ConnectionError/Timeout: print error, exit(1)
2. check_subscription() — on any error: warn, continue with local-only
3. If len(local) == 0: print pull hint, exit(1)
4. If len(local) == 1: auto-select, print notice, return name
5. Render Rich Table (Local group then Cloud group)
6. Prompt.ask("Enter number", default="1")
7. Validate input; on invalid: re-prompt once, then use default
8. Return selected model name
```

#### `with_fallback(model: str, chat_fn, ...) -> str`

Wraps the chat call. On HTTP 404 / model-not-found JSON error, prints warning and retries with `DEFAULT_MODEL`.

```python
def with_fallback(model: str, call_fn) -> str:
    try:
        return call_fn(model)
    except ModelNotFoundError:
        console.print(f"[yellow]Model {model!r} unavailable — falling back to {DEFAULT_MODEL}[/yellow]")
        return call_fn(DEFAULT_MODEL)
```

`ModelNotFoundError` is a local exception raised when Ollama returns HTTP 404 or a JSON body containing `"model not found"`.

---

## `src/main.py` — changes

```python
# Add at top:
from models import select_model, with_fallback

# In main(), before the chat loop:
selected_model = select_model(console)
console.print(Panel(f"[bold]MyAgent[/bold] — model: [cyan]{selected_model}[/cyan]", expand=False))

# Replace hardcoded MODEL in chat():
# chat() gains a `model` parameter instead of module-level constant
```

Remove the module-level `MODEL = "smollm2:135m"` constant — `models.py` owns `DEFAULT_MODEL`.

---

## Table render (Rich)

```
┌─ Available Models ─────────────────────────────────────────────────────────┐
│  #   Name               Type    Context   Best For          Status         │
│  ─   ────────────────   ─────   ───────   ───────────────   ────────────── │
│  1   smollm2:135m [*]   Local   8k        Quick tasks       [green]● Available[/]    │
│  2   llama3.2:3b        Local   128k      General chat      [green]● Available[/]    │
│  ─   ────────────────   ─────   ───────   ───────────────   ────────────── │
│      (No active subscription)   Cloud     —                 [yellow]○ Unavailable[/] │
└────────────────────────────────────────────────────────────────────────────┘
[*] default   Enter number to select [1]:
```

---

## Error handling matrix

| Condition | Behaviour |
|-----------|-----------|
| Ollama not running (ConnectionError) | Print "Ollama not reachable at localhost:11434. Start Ollama and retry." → `sys.exit(1)` |
| `/api/tags` timeout | Same as above |
| No local models | Print "No local models found. Run: ollama pull smollm2:135m" → `sys.exit(1)` |
| Subscription check fails | Warn "[dim]Cloud models unavailable[/dim]", continue local-only |
| Invalid selection input | Re-prompt once; fallback to default on second invalid |
| Model not found during chat | `with_fallback` → warning + retry with `DEFAULT_MODEL` |

---

## Tasks (implementation order)

- Step 1 — `src/models.py`: constants, `Model` dataclass, `_lookup_meta`
- Step 2 — `list_local_models()` + error handling
- Step 3 — `check_subscription()` stub
- Step 4 — `select_model()` table render + prompt
- Step 5 — `with_fallback()` + `ModelNotFoundError`
- Step 6 — Update `main.py` to wire `select_model` + `with_fallback`
- Step 7 — Unit tests: mock HTTP, test list/select/fallback paths
- Step 8 — Update `pyproject.toml` entry point if needed; manual smoke test

---

## Test plan

| Test | Approach |
|------|----------|
| `list_local_models` returns correct `Model` list | Mock `requests.get`, assert fields |
| `list_local_models` on ConnectionError | Mock raises `ConnectionError`, assert `SystemExit(1)` |
| `list_local_models` on empty response | Assert `SystemExit(1)` + pull hint |
| `select_model` single model auto-selects | Assert no prompt, returns only model name |
| `select_model` valid number input | Mock `Prompt.ask` → `"2"`, assert returns model[1].name |
| `select_model` invalid → default | Mock `Prompt.ask` → `"99"`, assert returns default |
| `with_fallback` on success | Assert original result returned |
| `with_fallback` on `ModelNotFoundError` | Assert fallback model used, warning printed |
