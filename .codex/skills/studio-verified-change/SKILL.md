---
name: studio-verified-change
description: Use when making a bounded, verified change in apps/studio. Enforces Studio conventions: read standards and AGENTS.md, understand the target first, plan and get approval before editing, make the smallest scoped change, add failing-then-passing Vitest coverage, verify with Corepack pnpm, and show the final diff.
---

# Studio Verified Change

Use this workflow for non-trivial `apps/studio` changes where correctness must be proven with focused tests.

## Workflow

1. Ground in repo rules.
   - Read `/standards/standards-code.md`.
   - For Studio TS/React work, read `/standards/standards-typescript.md`.
   - Read root `/AGENTS.md` and `apps/studio/AGENTS.md`.

2. Understand the target from first principles.
   - Trace the relevant page, layout, data module, state store, or utility before proposing edits.
   - Respect Studio boundaries: thin pages, feature components in `components/interfaces`, React Query for server/cache state, Valtio for interactive state, `data/fetchers.ts` for API calls, and `data/sql/execute-sql-query.ts` for database SQL.

3. Plan before editing.
   - Draft a bounded plan with exact files, approach, risks, and done criteria.
   - Do not edit until the user approves the plan.

4. Make the smallest scoped change.
   - Touch only approved files unless the implementation proves the plan incomplete; if scope must expand, stop and explain.
   - Preserve hosted vs self-hosted behavior under `IS_PLATFORM`.
   - Avoid unrelated refactors, formatting churn, and new abstractions.

5. Prove behavior with tests.
   - Prefer colocated Vitest tests for the changed utility or data logic.
   - Add or update tests so the new coverage would fail before the implementation and pass after it.
   - Keep tests focused on behavior, edge cases, and regression risk.

6. Verify with focused Vitest.
   - In this repo, `pnpm` is not on `PATH`; run Vitest through Corepack:
     ```bash
     corepack pnpm vitest --run <paths>
     ```
   - Use the smallest relevant path list, for example:
     ```bash
     corepack pnpm vitest --run lib/sql-event-parser.test.ts components/interfaces/SQLEditor/SQLEditor.utils.test.ts
     ```
   - If dependencies are missing, install from the lockfile with approval:
     ```bash
     corepack pnpm install --frozen-lockfile
     ```

7. Close the loop.
   - Show the test command and result.
   - Show the full diff for intended files.
   - Confirm `git status --short` only includes intended files, or explicitly call out pre-existing unrelated changes.
