# Goal 001: Maximize Metric X via Prompt Optimization

## Definition

- **Target metric:** `metric_x >= 0.80`
- **Data:** `<dataset>`, `<date or episode range>`
- **Evaluation split:** dev
- **Max iterations:** 20
- **Status:** IN_PROGRESS

## Results Summary

| # | Hypothesis | Variants | Best metric_x | Judge | Result |
|---|-----------|---------|--------------|-------|--------|
| 0 | Baseline — no constraints | all | 0.31 | 22 | REJECTED |
| 1 | Add explicit constraint A | variant_a, variant_b | 0.42 | 51 | PARTIAL |

## Current Best

Config: `h001`, Variant: `variant_a`, `metric_x`: 0.42, Judge: 51

---

## Hypothesis 0: Baseline

**Status:** REJECTED

**Reasoning:** Starting point. No constraints in prompt. Expect agent to ignore rules X and Y.

**What changed:** Nothing — pure baseline to establish floor metrics.

**Config:** `configs/goals/001/h000-baseline.yaml`

**Results:**
- `metric_x`: 0.31 ± 0.04 (train), 0.28 (dev)
- Judge: 22 — agent violated constraints X and Y on every trigger

**Conclusion:** Baseline confirms agent doesn't follow implicit rules. Next: add explicit constraint A and measure whether judge score improves.

---

## Hypothesis 1: Add Explicit Constraint A

**Status:** PARTIAL

**Reasoning:** H0 showed agent ignores constraint A entirely. Adding it explicitly to the prompt with a worked example should reduce violations.

**What changed:** Added constraint A with dollar/unit examples to `prompt.constraints`.

**Config:** `configs/goals/001/h001-variant_a.yaml`, `configs/goals/001/h001-variant_b.yaml`

**Results:**
- `metric_x`: 0.42 ± 0.06 (train), 0.39 (dev)
- Judge: 51 — agent follows constraint A more often but still fails on edge case E

**Conclusion:** Explicit constraint improved judge score by +29 points. Edge case E is the next bottleneck. Next: add a specific rule for edge case E.

---

<!-- Add Hypothesis 2 here after running H1 experiments -->
