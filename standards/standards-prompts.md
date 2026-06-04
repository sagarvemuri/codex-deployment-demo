---
description: Prompt engineering standards for writing LLM prompts — applies to all prompt templates, system prompts, and LLM instructions. Apply when writing, reviewing, or discussing prompts.
globs:
  - "**/*.j2"
---

# Prompt Engineering Standards v1.0.0

LLM prompts are code — versioned, reviewed, and evaluated. Quality is measured by eval results, not by how the prompt reads to a human.

---

## 1. Generalize, Don't Enumerate

Write instructions that teach the model how to reason about quality, not which specific failures to watch for. A list of four failure cases catches four cases. A generalizable principle catches the entire class.

The right structure depends on context — a decision tree, a rubric dimension, or a single well-motivated constraint. The test: could a novel input fail quality standards in a way your prompt doesn't cover? If yes, you've enumerated instead of generalized.

## 2. Motivate Every Constraint

Every constraint must carry its reason. The model uses the *why* to generalize to edge cases the *what* doesn't cover. A bare rule is a brittle rule.

```
# Weak
State only claims supported by the provided sources.

# Strong
State only claims supported by the provided sources. This output
feeds a fact-checking pipeline — unsourced claims are flagged as
errors and degrade system trust scores.
```

## 3. No Contradictions

A prompt must have a single, unambiguous instruction hierarchy. If two instructions could conflict, scope one to override the other. Frontier models follow instructions precisely — contradictions consume reasoning tokens on reconciliation rather than useful work.

## 4. Positive Instructions Over Prohibitions

Prefer positive instructions that define the target directly. Prohibitions only exclude points from the output distribution. Use prohibitions when a specific failure mode is common and hard to prevent through positive framing alone.

```
# Weak
Don't return the raw database rows to the user.

# Strong
Transform database rows into the response schema before returning.
The response schema enforces field naming, type coercion, and
omission of internal-only columns.
```

## 5. Fewer Instructions, Better Followed

Every instruction competes for the model's attention. As prompt length grows, compliance with any single instruction degrades. Before adding a new constraint, weigh the compliance cost it imposes on every existing constraint.

## 6. Fix the Instruction, Not the Volume

If the model doesn't follow an instruction, the instruction is unclear. Rewrite it for clarity rather than adding emphasis markers. Emphasis comes from structure and placement, not typographic intensity.

## 7. Examples Show Process, Not Just Outcomes

Examples are the most reliable way to steer output quality. Show the *reasoning process* that produces quality, not just the final output. An outcome-only example teaches pattern matching. A process example teaches generalization.

For non-trivial behaviors, include at least one contrast pair (strong vs. weak) to make the quality bar concrete. For reasoning models, process examples still calibrate the quality bar and output format — the model's internal chain-of-thought replaces any step-by-step procedure you might otherwise prescribe, but it still needs to see what "good" looks like.

## 8. Message Structure and Data Placement

System messages define persona, constraints, and instructions that persist across turns. User messages carry per-request data and the immediate task. Put reference data (schemas, context, documents) before the task instruction — models attend more reliably to instructions adjacent to the generation boundary.

## 9. Structure With Explicit Delimiters

For complex prompts that mix instructions, context, examples, and inputs, use explicit structural delimiters — XML tags, markdown headers, or named sections — to create unambiguous boundaries. Choose the convention that the target model handles best and stay consistent.

## 10. Let Reasoning Models Reason

For models with internal chain-of-thought (extended thinking, reasoning-mode models), specify goals and constraints — not thinking procedures. Prescribing step-by-step processes constrains the model's internal reasoning, which is more effective than any procedure we could write.

---

## Agentic Prompts

Additional rules for prompts that drive tool-calling agents, multi-step pipelines, or autonomous workflows.

### A1. Completion Criteria and Persistence

Agents need explicit stop conditions that require verifying context before acting and output quality before stopping. Without them, agents stop after producing a partial result that looks plausible. Define when the agent is done, and tell it to keep working until all criteria are satisfied rather than stopping at the first plausible result. Verification should check content against the success criteria the prompt defines — file-existence checks verify plumbing, not quality.

### A2. Parallel Tool Use

Frontier models execute independent tool calls in parallel. Explicitly encourage this for agents that read multiple files, run searches, or perform independent operations.

---

## Checklist

- [ ] Instructions generalize rather than enumerate specific cases
- [ ] Every constraint has a stated motivation
- [ ] No contradictions within the prompt or across included fragments
- [ ] Positive framing preferred; prohibitions only for stubborn failure modes
- [ ] Each instruction earns its place — adding it improves net compliance
- [ ] Emphasis from structure, not volume
- [ ] At least one contrast pair showing process for non-trivial behaviors
- [ ] Persistent context in system messages; per-request data in user messages
- [ ] Reference data before task instruction
- [ ] Explicit delimiters between content types in complex prompts
- [ ] Goals and constraints drive the prompt; reasoning left to the model

For agentic prompts, also verify:
- [ ] Explicit completion criteria with persistence and content-level verification
- [ ] Parallel tool use encouraged where applicable
