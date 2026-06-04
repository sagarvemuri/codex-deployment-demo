---
description: Evaluation principles — metrics, LLM judges, statistical rigor, traceability. Apply when writing, reviewing, or discussing evaluations and experiments.
---

# Evaluation Principles

Guilty until proven innocent. An implementation without a completed eval that measures its claimed behavior is incomplete code.

---

## Before Writing Any Eval

Stop. Before writing a single line of eval code, answer these questions honestly. Most wasted eval runs fail here — not in the implementation.

**1. What question does this eval answer?** State it in one sentence. Not "does the system work" but "does the extraction component identify named entities from unstructured text with sufficient recall to replace manual tagging?" If you can't state the question precisely, you're not ready to write the eval.

**2. Does the answer matter?** If this eval shows strong results, what decision does that justify — ship it, merge it, change the approach? If you can't name a concrete decision that hinges on the result, the eval is busywork. If weak results wouldn't change your plan either, the eval is theater.

**3. Could a bad system pass this eval?** This is the test for laxity. Imagine the laziest possible implementation that technically satisfies your metrics. If that system would score well, your eval is too easy — it's measuring the floor, not the ceiling. Tighten the rubric, use harder cases, or measure something closer to what actually matters.

**4. Does the eval mimic production?** Eval inputs must match the distribution, complexity, and messiness of real data. An eval on clean, curated examples tells you how the system performs on clean, curated examples — not on production traffic. If production has long documents, ambiguous cases, multilingual input, or adversarial formatting, the eval must too.

**5. What metrics directly capture the question?** Don't default to accuracy. Don't test proxies. If you care about entity extraction, measure entity recall — not JSON validity. If you care about factual correctness, measure fact coverage — not fluency.

---

## Core Rules

### Eval What the System Does

Metrics must directly measure the component's purpose. If you're comparing apples and oranges, don't test whether they're both red — test whether they're both fruits.

| System Type | Good Metrics | Bad Metrics |
|-------------|--------------|-------------|
| Classification | Precision, Recall, F1 | Exact string match |
| Extraction | Recall of expected entities | Character-level similarity |
| Generation | Key fact coverage, semantic similarity | Exact match |
| Search/Retrieval | Recall@K, MRR, NDCG | Simple presence check |

### Isolate Variables

When comparing systems or configurations, exactly one independent variable changes between conditions. Everything else is held constant: same model, same temperature, same data, same prompts. Without isolation, you cannot attribute differences to the thing you changed.

Every comparison needs a control condition — a clear baseline. Without one, results are uninterpretable.

Before trusting results, identify confounds: could something other than the variable you changed explain the difference? If yes, the experiment design needs work.

### No Data Leakage

Test data must not be used during development or tuning. If you've seen the eval cases while building the system, you've contaminated the signal. Generate eval sets independently and cache them before development begins.

### No Brittle String Parsing

Use LLM judges for semantic evaluation. Never exact string matching, substring checks, or regex for semantic content. For judge models, prefer fast and capable models (e.g., Gemini Flash) for high-volume evaluation. For complex evaluations requiring deep reasoning, use the strongest available model (e.g., Claude Opus).

### LLM-as-Judge

Judge prompts follow the prompt engineering standards — generalize rather than enumerate failure cases, motivate every rubric dimension.

Additional judge-specific requirements:

- Use a different model for judge vs generator
- Before trusting a judge, manually review its judgments yourself across a representative sample. If the judge disagrees with your own reading, the rubric is broken — fix it before using it at scale
- Randomize output order to mitigate position bias
- Be aware of verbosity bias (judges prefer longer outputs) and self-preference bias (models favor their own style) — design rubrics that resist these

Use deterministic checks where appropriate: schema validation, required field presence, categorical outputs, numerical bounds. Don't pay for an LLM call when the answer is unambiguous.

---

## Five Pillars of Test Coverage

Every eval must cover:

| Pillar  | Purpose |
|--------|---------|
| Happy paths | Common input variations |
| Failure modes | Graceful degradation |
| Edge cases | Boundary conditions |
| Adversarial | Inputs designed to break the system |
| Scale | Performance under load |

---

## Statistical Rigor

When comparing systems, you need to know whether the observed difference is real or noise.

- Report variance or standard deviation alongside every point estimate. A mean without spread is meaningless.
- Choose mean vs median deliberately — median resists outliers, mean captures magnitude. Report both when they diverge.
- Use appropriate significance tests (paired bootstrap, permutation tests) when comparing systems. The question is: "could this difference be explained by chance alone?"
- The required sample size depends on the effect size you're trying to detect. Small effects need more data, large effects need less. If you can't tell whether the difference is real or noise, you don't have enough data.
- Never pick an arbitrary threshold and call it "passing." That number is a fiction. Report the distribution, show per-case breakdowns, and let the evidence speak.

---

## Traceability and Data Capture

Capture everything. You should be able to trace any score back to the exact inputs, outputs, and intermediate steps that produced it.

- Every LLM call's full input and output is captured — not just the parsed or scored result
- Per-instance breakdowns are mandatory alongside aggregates. An eval that only reports "82% accuracy" hides the information you actually need.
- Errors and failures are recorded with full context, never silently skipped
- Metadata on every report: model, temperature, timestamp, eval set version, instance count. Two reports without this context are incomparable.
- Raw data is persisted so results can be re-analyzed without re-running

---

## Code Quality

Eval code follows code standards and language-specific standards. Additionally:

- No mocked LLM calls. `unittest.mock`, `MagicMock`, `patch`, `AsyncMock` on LLM calls are forbidden. Mocks produce fake signal — evals require real model behavior.
- Eval sets loaded from versioned JSON, not generated inline
- CLI interface with overridable parameters (`--limit`, `--seed`, `--model`, paths to eval set and report directory)

---

## Technical Requirements

1. **Parallelization**: Parallelize across both models AND cases (not just one axis).
2. **Caching**: Generate test data once, cache it. LLM-generated test cases MUST be cached for reproducibility.
3. **Reproducibility**: Random seeds fixed. Eval sets versioned. A report's metadata must be sufficient to reproduce the run.
4. **Latency tracking**: Every operation measured and logged.
5. **Structured output**: Each case yields a structured record containing model, task, case ID, success, latency, score, error, and full LLM input/output.
6. **Persistence**: JSON reports saved to `reports/` directory. Self-contained, shareable.
7. **Fast feedback**: Full suite runs in minutes, not hours.

---

## Eval Structure

```
scripts/
├── eval_[component].py
├── eval_cache/                # Cached test inputs
│   └── [component]_cases_v1.json
└── reports/                   # Eval outputs
    └── [component]_YYYYMMDD_HHMMSS.json
```

---

## After the Eval Runs

An eval is not a gate — it's evidence. It tells you what the system does, where it works, and where it breaks.

1. Using the captured per-case rows, identify failure patterns
2. Run end-to-end in the full pipeline context
3. Measure latency under production-like conditions
4. Validate integration points

Before trusting the results, answer honestly: if this eval shows strong results, would you trust that the system actually works as claimed? If it shows weak results, would you confidently say the system needs work? Can you trace any individual score back to the exact inputs, outputs, and intermediate steps that produced it? If any answer is "not really," the eval has gaps.

**If your eval looks clean but production fails, your eval is wrong.**

---

## Checklist

Before writing code:
- [ ] Eval question stated in one sentence with a concrete decision it informs
- [ ] Adversarial check: could a bad system pass this eval?
- [ ] Eval inputs match production distribution and complexity
- [ ] Metrics directly measure the component's purpose, not a proxy

Design:
- [ ] Exactly one independent variable between compared conditions
- [ ] Confounds identified and controlled for
- [ ] No data leakage — test data not seen during development
- [ ] LLM judge uses a different model than the generator
- [ ] Judge rubric reviewed against manual sample before trusting at scale
- [ ] Output order randomized for judge evaluations

Implementation:
- [ ] Parallelized across models and cases
- [ ] No mocked LLM calls
- [ ] Eval sets loaded from versioned JSON
- [ ] Every LLM call's full input and output captured
- [ ] Report metadata: model, temperature, timestamp, eval set version, instance count

Results:
- [ ] Variance / standard deviation reported alongside point estimates
- [ ] Per-instance breakdowns alongside aggregates
- [ ] Full suite runs in minutes
