---
description: Code standards — principles and rules for all code regardless of language
alwaysApply: true
---

# Code Standards v1.0.0

These rules apply to ALL code, regardless of language. Language-specific standards (Python, TypeScript) extend them.

---

## Principles

### Build for Scale

Every architectural decision assumes production scale from day one. The foundation built today must support 100x growth without rewriting. Design data models, interfaces, and pipelines for the volume and complexity you expect in a year — not for a prototype.

### Eval-Driven Development

Every new implementation must prove it works through real, isolated evaluation at scale — not toy tests or serialized model objects. The eval comes BEFORE the implementation is considered complete. See the eval standards for the full framework.

### Simple, Not Easy

The simplest code is the code that works — meaning it truly accomplishes the goal at hand. Complexity is added only when proven necessary.

### Code is a Liability

Code is not an asset; it's a maintenance burden. Ruthlessly delete, merge, and simplify.

---

## Rules

### 1. Fail Loudly — No Silent Fallbacks

Never silently swallow errors, return defaults on failure, or catch exceptions without rethrowing. Code must fail immediately at the point of error with full context: relevant input identifiers, exception type, and message. See language-specific standards for logging and examples.

### 2. Strict Type Safety

Define explicit types for all data structures. Use language-specific type systems (Pydantic models for Python, Zod for TypeScript). Validate at system edges only. Trust types after validation.

### 3. No Code Duplication (Strict DRY)

Search the codebase before implementing new functionality. Extract repeated patterns to shared utilities. Never copy-paste code between files or reimplement existing functionality.

### 4. No Adjectives in Naming

Never `EnhancedX`, `ImprovedY`, `BetterZ`, `NewX`, `V2`, `OptimizedX`. Names describe WHAT they do, not quality judgments.

```python
# NEVER
class EnhancedUserService: ...
def parseDataV2(): ...

# ALWAYS
class UserService: ...
def parseDataAsync(): ...  # Describes HOW
```

Exception: migration periods only (`UserServiceLegacy` → `UserService`), delete legacy immediately.

### 5. No Magic Values

Every hardcoded number or string gets a named constant.

```python
# NEVER
if password.length >= 8: ...

# ALWAYS
MIN_PASSWORD_LENGTH = 8
if password.length >= MIN_PASSWORD_LENGTH: ...
```

### 6. Single Responsibility

Each function does one thing well. If it's doing data fetching, transformation, validation, and side effects, it's doing too much. Split into focused, single-purpose units.

### 7. Parallelize Independent Operations

```python
# NEVER: 300ms total
result1 = await op1()  # 100ms
result2 = await op2()  # 100ms
result3 = await op3()  # 100ms

# ALWAYS: ~100ms total
result1, result2, result3 = await asyncio.gather(op1(), op2(), op3())
```

Parallelize: independent database queries, API calls, file operations. Don't parallelize: operations depending on previous results or sharing mutable state.

### 8. No Legacy Code

No old code paths "just in case." No backwards compatibility unless production demands it. No dead code. One implementation, the current one. Git history preserves what was deleted.

### 9. No *What* Comments — Yes *Why* Comments

Never comment what the code does — that's the code's job. Do comment why: trade-offs, constraints, non-obvious invariants, or design decisions that the code alone cannot convey. Block-level only, never inline.

### 10. Secrets in Env Vars, Config in Constants

Environment variables for: database URLs, API keys, tokens, service endpoints. Top-level constants for: business logic, timeouts, limits, thresholds, feature flags.

---

## Checklist

- [ ] No silent fallbacks — errors with full context
- [ ] Strict types throughout
- [ ] No code duplication — checked existing utilities
- [ ] No adjective naming
- [ ] No magic values — all constants named
- [ ] Single responsibility per function
- [ ] Independent operations parallelized
- [ ] No legacy/dead code
- [ ] No *what* comments; *why* comments where non-obvious
- [ ] Secrets in env vars, config in constants
