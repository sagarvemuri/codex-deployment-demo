# Codex Deployment Demo — Context System + Verified Change

A small demonstration of how I'd deploy Codex into a real enterprise monorepo: not "install it and start prompting," but stand up the **context system** that makes an agent consistent across a team, then prove the workflow on an actual change with tests.

I ran this on the [Supabase](https://github.com/supabase/supabase) monorepo — a genuinely complex, multi-language codebase (Next.js/React/TypeScript, Go, Postgres, Deno) — because toy apps don't surface the things that actually matter at enterprise scale.

The thesis: **the deployment is the context system; the model is table stakes.** Adoption stalls when every engineer re-teaches the agent the org's conventions from zero. The deployment engineer's job is to build the scaffolding that makes the agent inherit the team's taste automatically.

---

## What's here

### 1. Codified standards (`standards/`)
The base layer the agent reads on every task — universal code standards plus language-specific TypeScript, evaluation, and documentation standards. This is how the agent inherits the team's conventions instead of guessing.

### 2. AGENTS.md hierarchy
- `AGENTS.md` (root) — universal operating context: read the standards, understand from first principles, smallest scoped change, close the loop before declaring done.
- `apps/studio/AGENTS.md` (scoped) — package-local conventions for the Studio dashboard: thin pages / feature components / layouts, the single fetch boundary (`data/fetchers.ts`), the SQL execution boundary, React Query for server state vs Valtio for interactive state, and the hosted-vs-self-hosted split.

The scoped file **merges on top of** the root and overrides for that package — so each team gets the right local context without bloating one giant root file. (This also mitigates instruction-following decay: smaller, scoped context beats one file that the agent stops honoring deep in a session.)

### 3. A reusable Skill (`.codex/skills/studio-verified-change/`)
Encodes the exact diagnose → plan → change → verify loop as a repeatable workflow any engineer can invoke — including the repo-specific verification command. This is the unit of scaling a deployment past one person: the workflow becomes shared infrastructure, not tribal knowledge.

### 4. A verified change (`sql-event-parser-fix.patch`)
A real, bounded fix to Studio's SQL event parser, taken through plan-then-execute with test-first verification.

---

## The verified change — what actually happened

The change: harden quoted-identifier normalization in `apps/studio/lib/sql-event-parser.ts`. The original `cleanIdentifier` stripped all quote characters, so a Postgres identifier like `"user""table"` (an escaped double-quote) was mangled into `usertable` instead of the correct `user"table`.

The process is the point:

1. **Plan first.** Codex proposed a bounded plan scoped to three files, with named risks and explicit "done when" criteria — and crucially, tests written to *fail before and pass after*, so the fix is provably the cause.
2. **Execute against the locked plan.**
3. **Verify — and this is where it earned its keep.** Running the focused test suite surfaced a bug the plan hadn't anticipated: the table-side regex already handled escaped quotes, but the **schema-side capture didn't**. Rather than patch the symptom, the fix went upstream and aligned the schema capture pattern with the table capture across every pattern group.
4. **Result:** 140 tests pass.

If the loop had stopped at "did it make the edit?", a half-fix ships. The gap between *generated* and *verified* is the entire job — and it's why every workflow here closes with a real test run, not a vibe.

---

## Running it

The standards, AGENTS.md files, and Skill are dropped into a checkout of the Supabase monorepo. The fix is provided as a patch against that repo. Verification in this repo runs through Corepack (pnpm isn't on PATH by default):

```bash
corepack pnpm vitest --run \
  lib/sql-event-parser.test.ts \
  components/interfaces/SQLEditor/SQLEditor.utils.test.ts
```

---

## Why this shape

This mirrors how I'd actually onboard a customer team to Codex: start with the smallest high-value wedge, stand up the context system (standards → AGENTS.md hierarchy → Skills), and prove trust on bounded, verified changes before scaling to automations and parallel agents. Measure quality (revert rate, escaped-defect rate, eval pass rate), not just activity (PRs assisted) — because activity up with quality down kills trust faster than no tool at all.
