# agent-context-system

A reusable context system for running coding agents reliably on large, real-world monorepos. The idea: an agent is only as good as the context it operates in. Instead of re-teaching an agent your conventions on every task, you encode them once as shared infrastructure — codified standards, a scoped instructions hierarchy, and verified-change workflows the agent follows automatically.

I built and validated this on the [Supabase](https://github.com/supabase/supabase) monorepo because it's genuinely complex — multi-language (Next.js/React/TypeScript, Go, Postgres, Deno) and large enough that shallow context falls apart fast.

The thesis: **the context system is the work; the model is table stakes.** Most of the value in running coding agents at scale comes from the scaffolding around them, not the model itself.

---

## What's here

### 1. Codified standards (`standards/`)
The base layer the agent reads on every task — universal code standards plus language-specific TypeScript, evaluation, and documentation standards. This is how the agent inherits a consistent set of conventions instead of guessing per task.

### 2. Scoped instructions hierarchy (`AGENTS.md` files)
- `AGENTS.md` (root) — universal operating context: read the standards, understand from first principles, make the smallest scoped change, verify before declaring done.
- `apps/studio/AGENTS.md` (scoped) — package-local conventions for the Studio app: thin pages / feature components / layouts, the single fetch boundary, the SQL execution boundary, server-state vs interactive-state separation, and the hosted-vs-self-hosted split.

The scoped file **merges on top of** the root and overrides for that package — so each part of the repo gets the right local context without bloating one giant root file. This also mitigates instruction-following decay: smaller, scoped context holds up better than one file the agent stops honoring deep in a long session.

### 3. A reusable Skill (`.codex/skills/studio-verified-change/`)
Encodes a diagnose → plan → change → verify loop as a repeatable workflow, including the repo-specific verification command. Encoding the workflow once makes it reproducible across a team instead of living in one person's head.

### 4. A verified change (`sql-event-parser-fix.patch`)
A real, bounded fix to a SQL event parser, taken through plan-then-execute with test-first verification.

---

## The verified change — what happened

The change: harden quoted-identifier normalization in a SQL event parser. The original logic stripped all quote characters, so a Postgres identifier like `"user""table"` (an escaped double quote) was mangled into `usertable` instead of the correct `user"table`.

The process is the point:

1. **Plan first.** The agent proposed a bounded plan scoped to three files, with named risks and explicit "done when" criteria — including tests written to *fail before and pass after*, so the fix is provably the cause.
2. **Execute against the locked plan.**
3. **Verify — where it earned its keep.** Running the focused test suite surfaced a bug the plan hadn't anticipated: the table-side regex already handled escaped quotes, but the schema-side capture didn't. Rather than patch the symptom, the fix went upstream and aligned the schema capture pattern across every pattern group.
4. **Result:** 140 tests pass.

If the loop had stopped at "did it make the edit?", a half-fix ships. The gap between *generated* and *verified* is the whole point — which is why every workflow here closes with a real test run, not a vibe.

---

## Running it

The standards, `AGENTS.md` files, and Skill drop into a checkout of the target monorepo. The fix is provided as a patch against that repo. Focused tests run via the package manager declared by the repo:

```bash
corepack pnpm vitest --run \
  lib/sql-event-parser.test.ts \
  components/interfaces/SQLEditor/SQLEditor.utils.test.ts
```

---

## Why this shape

Adoption of coding agents tends to stall when every engineer re-teaches the agent the same conventions from scratch. This repo is a small, concrete answer to that: start narrow, encode the context system (standards → instructions hierarchy → Skills), and prove trust on bounded, verified changes before automating more. Measure quality (revert rate, escaped-defect rate, eval pass rate), not just activity — because activity up with quality down erodes trust faster than no tool at all.
