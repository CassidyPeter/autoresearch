# Auto Researcher Template

A template repository for autonomous AI scientist projects. Clone this to bootstrap a new research project where Claude Code (Opus) runs hypothesis-driven experiments end-to-end — no human in the loop.

## Why This Structure Exists

Running AI experiments without structure gets messy fast. After a few iterations you end up with scattered configs, unnamed result files, and no record of what was tried or why. It becomes impossible to tell which change caused which outcome — and the model has no context to build on, so it repeats mistakes or explores in circles.

This template imposes a goal-oriented structure so that:

- **Every experiment is traceable.** Results are stored under `goal → hypothesis → variant → run`, mirroring the config and doc structure. You always know what was tested and what it produced.
- **The model can iterate intelligently.** Each hypothesis is documented with its reasoning and conclusion. Claude reads this before forming the next hypothesis — it adapts based on evidence rather than guessing blind.
- **Nothing gets lost between sessions.** The goal doc is the source of truth. Whether Claude resumes after 10 minutes or 10 days, it picks up exactly where the last hypothesis ended.
- **Progress is visible at a glance.** `docs/goals/index.md` shows every goal, its current status, and best result so far — no digging through files.

## What This Is

This repo defines a **doc and directory structure** for projects where an AI scientist:

1. Reads a goal
2. Forms a hypothesis
3. Runs experiments
4. Analyzes results
5. Iterates until the goal is met

The structure is domain-agnostic — it works for any project where you're optimizing an AI agent's behavior through systematic backtesting or simulation.

## Repository Structure

```
CLAUDE.md                        ← entry point for Claude (read this first)
configs/
  baseline.yaml                  ← default experiment parameters
  goals/
    001/                         ← one dir per goal
      h000-baseline.yaml         ← one config per hypothesis × variant
      h001-variant_a.yaml
docs/
  goals/
    index.md                     ← all goals and their status
    001-example-goal.md          ← goal definition + hypothesis log
  experiment_runbook.md          ← scientist loop, file formats, CLI reference
  technical/
    architecture.md              ← codebase layers and file reference
    runbook.md                   ← how to run, debug, and inspect results
    compute_guide.md             ← server setup and resource estimates
results/
  goals/
    001/
      h000/                      ← results per hypothesis
        variant_a/
          run0/
            result.json          ← metrics 
```

## How to Use This Template

### 1. Clone and rename

```bash
git clone <this-repo> my_project
cd my_project
```

### 2. Ask Claude Code to change documents according to your project

Open Claude Code in this directory and tell it what you want to build. The docs provide a full spec — Claude will read `docs/technical/architecture.md` and implement the framework for your domain.

## Key Conventions

| Convention | Meaning |
|-----------|---------|
| `hNNN` | Hypothesis ID — same across goal doc, config filename, and results dir |
| `NNN` | Goal ID — zero-padded 3-digit number |
| `train` split | Used for tuning; agent sees this data |
| `dev` split | Held out; only checked after training converges |
| 6 parallel runs | Default to reduce randomness in LLM outputs |
| Judge model | Always `anthropic/claude-sonnet-4-6` — do not change unless stated in goal |

## Scientific Discipline

- **One hypothesis at a time.** Don't batch 10 configs before seeing results.
- **Base each hypothesis on evidence.** What did the last experiment reveal?
- **Don't touch dev until train is tuned.** Dev is for final validation, not optimization.
- **Document conclusions.** Every hypothesis in the goal doc needs a conclusion before moving on.
