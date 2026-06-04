---
description: LLM model selection — approved model shortlist and usage guidance. Apply when selecting models, writing LLM calls, or reviewing model choices.
---

# Model Standards v1.0.0

## Approved Model Shortlist

Use only models from this table. Maintain a consistent identifier format (e.g., `provider/model`) across all code and configuration.

| Model | Tier | Use When |
|-------|------|----------|
| `anthropic/claude-opus-4.6` | Deep reasoning | Complex evaluation, nuanced judgment, hard problems |
| `openai/gpt-5.4` | Deep reasoning | Complex evaluation, multi-step reasoning |
| `anthropic/claude-sonnet-4.6` | Medium | General-purpose tasks, balanced cost/quality |
| `openai/gpt-5.4-mini` | Medium | General-purpose tasks, balanced cost/quality |
| `google/gemini-3.1-pro-preview` | Medium | General-purpose tasks |
| `google/gemini-3-flash-preview` | Fast | High-volume, low-latency |
| `anthropic/claude-haiku-4.5` | Fast | High-volume, low-latency |
| `x-ai/grok-4.1-fast` | Fast | High-volume, low-latency |

### Embeddings

| Model | Use When |
|-------|----------|
| `openai/text-embedding-3-large` | Default embedding model |

## Selection Rules

1. **Deep reasoning tasks** (analysis, probability estimation, complex evaluation): `anthropic/claude-opus-4.6`, `openai/gpt-5.4`
2. **General tasks** (structured output, moderate reasoning): `anthropic/claude-sonnet-4.6`, `openai/gpt-5.4-mini`, `google/gemini-3.1-pro-preview`
3. **High-volume tasks** (instruction following, bulk classification, fast extraction): `google/gemini-3-flash-preview`, `anthropic/claude-haiku-4.5`, `x-ai/grok-4.1-fast`
4. Use only models from the shortlist. Do not substitute deprecated IDs (`gpt-4.1-mini`, `gpt-4o`, `claude-3.5-sonnet`, `gemini-2.0-flash`, `openai/o3`, etc.).
5. **Eval scripts**: use the model appropriate for the task being evaluated. When testing deep reasoning, use a deep reasoning model.

## Updating the Shortlist

When new models are released, evaluate them against the existing shortlist before adding. Remove deprecated models promptly. The shortlist should reflect the best available models at each tier — not accumulate historical entries.

---

## Checklist

- [ ] Model ID uses a consistent `provider/model` format from the shortlist
- [ ] Tier matches the task complexity
- [ ] Judge model differs from generator model (for evals)
- [ ] No deprecated or unlisted model IDs
