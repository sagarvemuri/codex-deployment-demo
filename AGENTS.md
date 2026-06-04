# AGENTS.md — Operating Context

## Before any task
Read the relevant files in `/standards` and follow them. `standards-code.md`
is universal; apply `standards-typescript.md` for TS/React work and
`standards-eval.md` before trusting any change.

## This repo
Supabase monorepo. Key surfaces: `apps/studio` (Next.js/React/TS dashboard),
the Go auth service, Postgres tooling, Deno edge functions.

## How to work here
- Understand a subsystem from first principles before changing it.
- Make the smallest change that achieves the goal; no unrelated refactors.
- Close the loop: run the relevant tests / verify before declaring done.
- Keep context scoped — prefer per-package AGENTS.md over one bloated root file.