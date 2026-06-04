---
description: Python standards — Pydantic, LLM client patterns, uv run, structured logging
globs:
  - "**/*.py"
---

# Python Standards v1.1.0

These rules are mandatory for all Python code. See the code standards for universal principles.

---

## 1. Pydantic Models Over Runtime Type Checking

Use Pydantic models with strict types for all data structures. Avoid `isinstance` checks that indicate missing type discipline — if you're branching on type at runtime, the data model is wrong.

```python
# NEVER — branching on type means the model is underspecified
if isinstance(result, dict):
    process_dict(result)
elif isinstance(result, list):
    process_list(result)

# ALWAYS — the type system carries the information
class Result(BaseModel):
    items: list[ResultItem]
```

**Permitted uses of `isinstance`:** union narrowing after Pydantic validation, protocol detection in generic utilities, and type guards that the type checker requires. **Permitted uses of `getattr`/`hasattr`:** accessing optional attributes with defaults in generic code, protocol checking. These are tools — use them when the type system alone can't express the constraint, not as substitutes for modeling your data.

`Any` may exist only transiently within edge input parsing functions whose sole purpose is to sanitize external data into Pydantic models.

## 2. Error Handling — Full Context

Follow general fail-loud rules; in Python, log with structured logging before re-raising when structured context is needed.

```python
import structlog

logger = structlog.get_logger()

def parse_data(input: str) -> ParsedData:
    try:
        return json.loads(input)
    except json.JSONDecodeError as e:
        logger.error(
            "Failed to parse JSON",
            input_preview=input[:100],
            error=str(e),
        )
        raise ValueError(f"Invalid JSON input: {e}") from e
```

## 3. File Structure

Modules start as a single `core.py` with models inline. Split into the four-file structure when `core.py` exceeds ~300 lines or when models are shared across modules:

```
modules/MODULE_NAME/
├── __init__.py      # Public API exports
├── core.py          # Main business logic
├── models.py        # Pydantic models, data structures
└── utilities.py     # Helper functions (if needed)
```

No file should exceed ~500 lines. Production code in `modules/` (or `src/`), experimental code in `scripts/`.

## 4. Shared Modules (Mandatory)

Establish and use shared utilities for cross-cutting concerns. Never reimplement what already exists. Common shared modules include:

| Concern | Approach |
|---------|----------|
| Structured logging | `structlog` or equivalent — never `print()` or `warnings.warn()` |
| Database access | Shared connection pool and driver utility |
| Environment config | Centralized env var management (e.g., `pydantic-settings`) |
| HTTP client | Shared HTTP client with retry/timeout defaults |
| Embeddings | Shared embedding client utility |
| Prompt rendering | Jinja2 template loader |

Never use `print()`, `warnings.warn()`, or bare `import logging` for application logging. Never reimplement shared utilities. If they don't work, extend them without breaking existing use cases.

## 5. Execution

```bash
# ALWAYS — use a project-level runner
uv run python -m myproject.modules.feature.core

# NEVER — bare Python invocation
python file.py
```

Use `uv run` (or your project's equivalent task runner) to ensure consistent dependency resolution and virtual environment usage.

## 6. LLM Client Usage

Use a single, consistent LLM client library across the project — no custom wrappers. Whether you use LiteLLM, the provider SDKs directly, or an internal routing layer, the rules are the same.

### Basic Usage with Structured Output

```python
from pydantic import BaseModel

class Answer(BaseModel):
    result: int
    explanation: str

async def process():
    response = await llm_client.query(
        messages=[{"role": "user", "content": "What is 9 + 10?"}],
        model="google/gemini-3-flash-preview",
        response_format=Answer,
    )
    return Answer.model_validate_json(response.content)
```

### Multi-Turn Conversations

```python
messages = [{"role": "user", "content": "What is 9 + 10?"}]
response = await llm_client.query(messages=messages, model=MODEL)
messages.append({"role": "assistant", "content": response.content})
messages.append({"role": "user", "content": "Elaborate?"})
response = await llm_client.query(messages=messages, model=MODEL)
```

### Tool Calls

```python
tools = [
    {
        "name": "get_weather",
        "parameters": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"],
        },
        "description": "Get current weather for a location",
    }
]

response = await llm_client.query(messages=messages, model=MODEL, tools=tools)
if response.tool_calls:
    for tc in response.tool_calls:
        result = get_weather(json.loads(tc.arguments)["location"])
        messages.append({"role": "tool", "tool_call_id": tc.id, "content": result})
    response = await llm_client.query(messages=messages, model=MODEL)
```

### Embeddings

```python
response = await llm_client.embed(
    text="Sample text",
    model="openai/text-embedding-3-large",
)
vector = response.embedding
```

### Caching

Cache LLM responses to disk during development so you can iterate on pipeline code without repeated API calls. Use a caching layer appropriate to your client library.

**Key rules:**
- JSON validation handles retries automatically — if it fails, your prompt is wrong
- Always use Pydantic models for structured outputs
- No manual JSON parsing of LLM responses
- Error handling must include model identifier and request context

---

## Checklist

- [ ] Pydantic models with strict types — `isinstance`/`getattr`/`hasattr` only for union narrowing, protocol checks, or type guards
- [ ] Single `core.py` for small modules; four-file split when >300 lines or models shared; no file >500 lines
- [ ] Uses shared utilities — no reimplementation of logging, HTTP, database, or config
- [ ] `uv run` (or equivalent task runner) only
- [ ] Single LLM client — no wrappers, no manual JSON parsing
- [ ] Model IDs from the approved shortlist (see model standards)
- [ ] Error handling with full context via structured logger
