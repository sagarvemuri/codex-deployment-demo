---
description: Documentation standards — handoffs, findings, agent guides, roadmaps, and all markdown documentation
globs:
  - "**/*.md"
  - "**/HANDOFF*"
  - "**/FINDINGS*"
  - "**/ROADMAP*"
---

# Documentation Standards

These rules apply to the four named document types (HANDOFF, FINDINGS, AGENTS, ROADMAP) and govern the core principles for all markdown documentation. Other markdown files (READMEs, design docs, changelogs, ADRs) follow the core principles (rules 1-9) but are not required to use one of the four skeletons. Code comments are governed by the code standards, not this document.

---

## 1. Structural Addability

Every document must be organized so that when new information arrives, the location to add it is unambiguous. A reader should never wonder "where does this go?" and a writer should never need to create a new section for something that already has a home.

This means:
- Every section has a clearly scoped purpose. The section heading and its first line make the scope obvious.
- Sections are mutually exclusive — no two sections accept the same kind of information.
- If information doesn't fit any existing section, that's a signal to reconsider whether it belongs in this document at all.

**Test:** Before adding content, identify the exact section it belongs in. If you can't, the content either doesn't belong or the document structure is broken.

## 2. No Process Noise

Document the work product, not the process of creating it. Implementation bugs, debugging detours, IDE issues, refactoring steps, and transient errors are NOT documentation. They are process artifacts that die with the session.

```markdown
<!-- NEVER -->
We initially used print statements instead of logger, which caused test failures.
After fixing the import, the eval ran successfully.
The first attempt failed because the API key was missing from .env.

<!-- ALWAYS -->
Eval results (script: `scripts/eval_position_bias.py`,
report: `reports/position_bias_20260409_105648.json`):

| Config | Brier | ECE |
|---|---|---|
| 1-perm | 0.085 | 0.183 |
| 2-perm | 0.072 | 0.108 |
```

**The filter:** Before writing any statement, ask: "Would this matter to someone who didn't watch me work?" If no, it's process noise. Delete it.

Things that ARE documentation: what the system does, what was measured, what the measurements were, what decisions were made, what remains to be done.

Things that are NOT documentation: what broke during development, what you tried before it worked, what typo caused a failure, what dependency was missing, what workaround you used temporarily.

## 3. No Interpretive Bias

Documentation reports. The reader interprets.

When presenting evaluation results, experimental data, or any measurements: state the question, state the method, state the numbers. Do not explain what the results "mean." Do not suggest what should be done next. Do not editorialize about whether a result is "good" or "bad" or "surprising" or "interesting." The data speaks.

```markdown
<!-- NEVER -->
The results show that forced-choice is significantly better, which means
we should use it in production. This is surprising because we expected
tuple to win. The key takeaway is that position bias matters more than
we thought.

<!-- ALWAYS -->
| Method | Brier | ECE | Accuracy |
|---|---|---|---|
| Forced-choice | 0.082 | 0.163 | 77.2% |
| Tuple stated | 0.104 | 0.175 | 74.4% |

Paired bootstrap significance (5000 resamples): p = 0.0004.
```

The reader decides what's surprising, what's important, and what to do next. The document's job is to make that decision possible by presenting complete, unbiased data.

**Statistical uncertainty is not hedging.** Confidence intervals, p-values, standard deviations, and sample sizes are measurements, not opinions. "95% CI [0.066, 0.098]" is data. "Probably around 0.08" is hedging.

**Where interpretation lives:** If the team has made a decision based on data, that decision is documented as a fact with attribution — not as an interpretation of results. "The team chose tuple extraction for production (2026-04-09, informed by Experiment: Platt Scaling)" is a fact that cites its source. "The results suggest tuple is better for production" is interpretation.

## 4. Full Traceability

Every claim has a verifiable source. The reader must be able to trace any statement back to the artifact that produced it without asking anyone.

| Claim type | Required source |
|---|---|
| Measurement / metric | Report file path + dataset size |
| Behavioral observation | Script path + input parameters |
| Decision | Date + who decided (or conversation/meeting ID) |
| External fact | URL or document path |
| Code behavior | File path + function name |

```markdown
<!-- NEVER -->
Position bias cancels cleanly with averaging.

<!-- ALWAYS -->
Position bias cancels with averaging — fair-coin probe averages to 50.12% / 49.88%
(script: `scripts/eval_position_bias.py`,
report: `reports/position_bias_20260409_105648.json`, n=3 probes).
```

## 5. Eval Results Require Review

Unreviewed evals are unverified data. Do not document eval results unless the evaluation has been formally reviewed — either through a dedicated review process, peer review, or a structured review checklist. Establish and follow a consistent review workflow before trusting eval outputs as evidence.

## 6. Tables for Quantitative Data

Numbers in prose are hard to compare. Any time you present 3+ related data points, use a table. Label columns precisely — include units, conditions, and sample sizes.

## 7. Point, Don't Paste

Reference files by path. Don't paste code blocks longer than 5 lines — give the reader a pointer and let them explore. Documentation is a map, not a photocopy.

## 8. Delete, Don't Annotate

When information becomes stale, rewrite or remove it. Never leave correction layers ("UPDATE: this is no longer true," "NOTE: superseded by..."). The document reflects current state at all times.

**Exception:** Experiment sections in FINDINGS.md are immutable records — see the FINDINGS.md rules below.

## 9. One Document, One Job

Each document type owns a specific kind of content. Every piece of information has exactly one home — the document whose purpose matches. Other documents reference it; they do not reproduce it.

| Content | Owned by | Other docs do |
|---|---|---|
| Experimental measurements, metrics, per-case breakdowns | FINDINGS | Link to FINDINGS experiment section |
| Observations, known limitations | FINDINGS | Link to FINDINGS |
| Decisions (with date and attribution) | FINDINGS | Link to FINDINGS decision |
| Prioritized work items, effort estimates, blockers | ROADMAP | Link to ROADMAP |
| Completion records (date + report path) | ROADMAP | Link to ROADMAP |
| Work transfer context, validation actions, open questions | HANDOFF | Link to HANDOFF |
| Directory layout, conventions, commands | AGENTS | Link to AGENTS |

**Test:** If you're about to write content, ask which document type owns it. If you're in the wrong document, write a link instead. If you can't identify an owner, the content either doesn't belong in documentation or the document set is incomplete.

---

## Document Types

### HANDOFF.md — Work Transfer

**Purpose:** Give the next person everything they need to continue the work — and nothing they don't. A cold reader should orient in under 60 seconds.

**Location:** In the module or feature directory where the work lives.

**Skeleton:**

```
# [Topic] — Handoff

## Status
One line. Current state, not history.

## Goal
What "done" looks like. One or two sentences.

## Context
Files, systems, tables, tools involved. Paths and names — not contents.
Links to PRs, issues, conversations, FINDINGS.md, related docs.

## What Still Needs Validation
Concrete actions. Each item specifies what to run, what to check,
and what dataset or environment to use.

## Open Questions
Specific, answerable questions. Each one names where to look
or what to try to get the answer.
```

**Rules:**
- Status is ONE sentence reflecting now, not the journey.
- Context is pointers: file paths, URLs, job IDs. The reader explores — you provide the map.
- Context links to FINDINGS.md for results. The handoff does not reproduce or summarize experimental data.
- Every validation item is a concrete action: "Run `eval_platt_scaling.py` on production scenarios from the last 30 days" — not "We should probably validate on real data."
- **Validation vs Open Questions:** If it's runnable, it belongs under Validation. If it requires investigation or a decision, it belongs under Open Questions.

### FINDINGS.md — Research Record

**Purpose:** The complete, objective record of what was built and what experiments measured. This is a lab notebook — it records observations, not opinions.

**Location:** In the module or experiment directory.

**Skeleton:**

```
# [Topic] — [Descriptive Subtitle]

## What We Built
One paragraph. What the system does, its primitives, its constraints.
Table of capabilities if there are multiple components.

## Observations
Numbered list. Each item states a measured behavior with its evidence
and source citation. No qualitative judgment ("good", "bad", "better
than expected"). What the system does when measured.

## Known Limitations
Numbered list. Each limitation:
- **What:** One-sentence description of the limitation.
- **Evidence:** Measurements or observations with source citations.
- **Conditions:** When this limitation manifests (input types, scale, etc.).
- **Status:** Not addressed / Partially addressed / Addressed.

**Observations** = what the system does. **Limitations** = which goals
or use cases are ruled out (or risky), with the same evidence standard.

## Experiment: [Name] ([Date])
Repeat for each experiment. Every experiment has exactly three parts:

- **Question:** What we are measuring. One sentence.
- **Method:** Script path, dataset, configuration, sample size.
  Enough detail to reproduce the run.
- **Results:** Tables. Raw measurements. No prose interpretation.
  Include statistical significance where applicable (test type,
  n, p-value).

That's it. No "Interpretation." No "Conclusions." No "Key Takeaways."
The results section is data. The reader interprets.

## Decisions
Dated list of decisions made based on the data above. Each entry:
- **[Date]:** What was decided. Link to the experiment or data
  that informed it. Who decided (person or team).
```

**Rules:**
- **Experiment sections are immutable records.** Once written, an experiment section is never edited or deleted. It is a permanent record of what was measured at that point in time.
- **Re-runs are additive.** If an experiment is re-run (same question, different method or parameters), add a new experiment section with a new date. Do not replace the original. Both records stay. A reader comparing the two experiments should be able to understand any difference in results from the differing questions, methods, or conditions — without any prose reconciling them.
- **Experiments are self-contained.** Each experiment section stands alone. A reader can understand one experiment without reading the others. If two experiments measure the same thing differently, each one's Question and Method make the difference clear. No "bridging" section explaining why results differ.
- Always include dates in experiment headers. Findings accumulate over time.
- Results sections contain tables, statistical tests, and source references. Nothing else.
- The Observations section reports what was measured, not why it's good or bad.
- Known Limitations are not bugs. They are measured behaviors that constrain the system's usefulness for a specific purpose.
- Decisions are the ONLY place where human judgment appears, and they are explicitly labeled with date and attribution. Decisions cite the experiment or data that informed them (by section link), but do not restate or interpret the data.
- **Eval results require review.** No experiment section may be added unless the evaluation has been formally reviewed — through peer review, a structured checklist, or a dedicated review process.

### AGENTS.md — Directory Guide

**Purpose:** Orient an agent or developer entering this directory. What's here, what the conventions are, where to start.

**Location:** Root of each major directory (repo root, app directories, module directories).

**Skeleton:**

```
# [Directory/Module Name]

Brief intro — what this directory is and does. One to three sentences.

## [Standards / Rules]
Links to applicable standards. Not restated — linked.

## Codebase Layout
Table mapping directories to purposes.

## Common Tasks
Table mapping tasks to skills, commands, or entry points.

## Quick Commands
Code block with the most common operations.
```

**Rules:**
- AGENTS.md files are scoped to their own directory. Don't duplicate parent AGENTS.md content.
- Tables over prose. "How do I run tests" is found in a table cell, not buried in a paragraph.
- Link to skills and docs — don't reproduce their contents.
- Keep under 100 lines. If longer, content belongs in a separate doc.

### ROADMAP.md — Future Plans

**Purpose:** What's planned, in what order. A prioritized list of work with enough context to understand ordering.

**Location:** In the module or project directory where the work will happen.

**Skeleton:**

```
# [Topic] — Roadmap

## Current State
One paragraph. Where things stand today. Link to FINDINGS.md or
HANDOFF.md for details.

## Priorities
Ordered list. Each item:
1. **[Name]** — One-sentence description.
   Dependencies or blockers. Expected effort.

## Done
Items moved here when complete. Date and link to PR or report.

## Deferred
Items explicitly deprioritized. Each with a one-sentence reason.
```

**Rules:**
- Priorities are ordered. Position 1 is next. Ties are indecision — pick one.
- Done items stay in the doc. They show progress and prevent re-proposing completed work.
- Deferred items have reasons. "Blocked by X" or "Lower impact than Y" — not "We decided not to."

---

## Writing Style

| Rule |
|---|
| Active voice, present tense |
| Short sentences — if it needs a semicolon, it needs two sentences |
| Parallel structure in lists |
| Bold for emphasis, not italics or ALL CAPS |
| One blank line between sections |
| No emojis |

---

## Common Anti-Patterns

These are the violations Rules 1–9 don't make obvious enough:

| Anti-Pattern | Rule Violated |
|---|---|
| Summary / recap sections at the end of a document | Structural addability (Rule 1) — creates a second source of truth |
| Reconciliation prose between experiments ("Experiment 3 differs from 1 because...") | FINDINGS immutability — each experiment is self-contained |
| Correction layers ("UPDATE: this is no longer true") | Delete, don't annotate (Rule 8) |

---

## Checklist

- [ ] Every section's content matches that section's stated scope (Rule 1: addability)
- [ ] No process noise — debugging steps, transient errors, implementation detours (Rule 2)
- [ ] No interpretive language in results — means, suggests, implies, shows that (Rule 3)
- [ ] Every claim has a verifiable source — file path, report, script, conversation ID (Rule 4)
- [ ] Eval results formally reviewed before documenting (Rule 5)
- [ ] Quantitative data in tables, not prose (Rule 6)
- [ ] No pasted code blocks longer than 5 lines — paths referenced instead (Rule 7)
- [ ] No stale content — everything reflects current state, except immutable experiments (Rule 8)
- [ ] Document follows the correct type skeleton (Rule 9)
- [ ] No hedging words — seems, likely, probably, might, could be
