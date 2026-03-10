# Experiment Runbook

## System Overview

Claude Code (Opus) acts as a scientist: sets a goal, forms hypotheses, runs experiments via CLI, analyzes results, and iterates until the goal is met.

## File Structure

```
docs/goals/
├── index.md                           # all goals overview
├── 001-example-goal.md                # goal definition + all hypotheses inline

configs/goals/
├── 001/
│   ├── h000-baseline.yaml             # one YAML config per hypothesis+variant
│   ├── h001-variant_a.yaml
│   └── h002-variant_b.yaml

results/goals/001/
├── h001/
│   ├── result.json                    # aggregated across all variants
│   ├── variant_a/
│   │   ├── result.json                # aggregated across runs
│   │   ├── run0/
│   │   │   ├── result.json            # single run: metrics + judge
│   │   └── run1/
│   │       └── ...
│   └── variant_b/
│       ├── result.json
│       └── run0/
```

Hierarchy: **goal → hypothesis → variant → run**. Claude drills down only when needed.

## Goal Document Format (`docs/goals/001-*.md`)

```markdown
# Goal: <One-line description>

## Definition
- Target metric: <metric_name> >= <threshold>
- Data: <dataset>, <date range or episode range>
- Evaluation split: dev
- Max iterations: 20
- Status: IN_PROGRESS

## Results Summary
| # | Hypothesis | Variants | Best <metric> | Judge | Result |
|---|-----------|---------|--------------|-------|--------|
| 0 | Baseline  | all     | -1.5%        | 15    | REJECTED |
| 1 | Add constraints | variant_a, variant_b | +3.2% | 62 | PARTIAL |

## Current Best
Config: h001, Variant: variant_a, <metric>: +3.2%, Judge: 62

## Hypothesis 0: Baseline
**Status:** REJECTED
**Reasoning:** Baseline shows agent ignores constraints X and Y. Explicit rules should fix it.
**What changed:** Added explicit constraint to prompt.
**Config:** configs/goals/001/h000-baseline.yaml
**Results:** <metric> -1.5%, Judge 15. Agent still ignored constraint.
**Conclusion:** Explicit rules alone don't work. Next: try worked step-by-step examples.

## Hypothesis 1: ...
```

IMPORTANT: Base your hypothesis on the result of the previous one. Do not create 10 hypotheses at once. Change your thinking according to new data from experiments.

Each hypothesis has: Status (PENDING/RUNNING/CONFIRMED/REJECTED/PARTIAL), Reasoning, What changed, Config path, Results, Conclusion.

## Result Files

**Hypothesis-level** (`h001/result.json`) — quick overview across variants:
```json
{
  "hypothesis": "h001",
  "goal_id": "001",
  "timestamp": "2026-01-01T00:00:00",
  "variants": {
    "variant_a": {
      "model": "openai/gpt-5-mini",
      "n_runs": 6,
      "train": {
        "metrics_avg": {"total_return": 0.032, "sharpe_ratio": 1.48},
        "metrics_std": {"total_return": 0.012},
        "judge_avg": 62, "judge_std": 7.1
      },
      "dev": {}
    }
  }
}
```

**Variant-level** (`h001/variant_a/result.json`) — aggregated metrics with std across runs, full judge for first run.

**Run-level** (`h001/variant_a/run0/result.json`) — single run metrics + full judge (score, reasoning, violations with step index/time). Both train and dev splits included.

## CLI Commands

```bash
# Run experiment with goal tracking
python3 -m <package> run --config configs/goals/001/h001-variant_a.yaml \
  --goal 001 --hypothesis h001 --variant variant_a

# Run standalone (flat output, no goal tracking)
python3 -m <package> run --config configs/baseline.yaml

# Run multiple experiments in parallel
python3 -m <package> run-all --configs-dir configs/goals/001/ --workers 4 --runs 6

# Inspect results
python3 -m <package> list
python3 -m <package> compare exp_a exp_b
python3 -m <package> trace exp_name --split dev    # print agent reasoning
python3 -m <package> fetch --config configs/baseline.yaml
```

## Variant Catalogue

Defined in `variants.py`. List all available variants with `python3 variants.py`.

Each variant defines:
- A prompt (strategy + constraints + tone)
- Which placeholders it uses (`state`, `indicators`, `history`, `context`)
- Which dataset/symbol it targets

The `h001` naming convention is the glue — same ID across docs, configs, and results. Each hypothesis can test one or many variants with multiple parallel runs.
