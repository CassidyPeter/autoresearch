# Architecture

> This document is a **spec for Claude Code to implement**. Point Claude at this file and describe your domain — it will build the framework from scratch.

## Overview

Layered system where each layer communicates only through its Protocol interface. The agent never knows it's in a simulation — it just sees state and tools.

```
configs/YAML
    ↓
experiment/runner.py      ← orchestrates everything
    ├── data/sources/     ← fetch + cache environment data
    ├── simulation/engine ← replay episodes, call agent at intervals
    │       └── agent/core_agent  ← LLM loop + tools
    └── evaluation/       ← objective metrics + LLM judge
    ↓
results/JSON + index.json
```

---

## File Reference

### `<package>/data/`

| File | Purpose |
|------|---------|
| `models.py` | Core data types: `Observation`, `Action`, `Episode`, `StateSnapshot` |
| `protocols.py` | `DataSource` Protocol — the only interface any data source must implement |
| `indicators.py` | Pure-function derived features from raw observations |
| `cache.py` | Cache keyed by `sha256(source+params)`, stored in a local cache dir |
| `sources/<source>.py` | Concrete data source implementations |

### `<package>/agent/`

| File | Purpose |
|------|---------|
| `llm_client.py` | OpenRouter wrapper (openai-compatible). Falls back to `fallback_model` on error. |
| `protocols.py` | `AgentRun`, `AgentStep`, `ToolCall` dataclasses — the agent output format |
| `placeholders.py` | Builds the system prompt from current state: fills `{strategy}`, `{constraints}`, `{state}` |
| `core_agent.py` | Core LLM loop (max N iterations). Dispatches tool calls, appends results, loops until done. Returns `AgentRun`. |
| `tools/` | Tool definitions (or inline in `simulation/engine.py` to close over simulator state) |

### `<package>/simulation/`

| File | Purpose |
|------|---------|
| `simulator.py` | Virtual environment: applies actions, tracks state, enforces constraints, takes snapshots. |
| `splitter.py` | Splits episode data into train/dev by date or index range |
| `engine.py` | Main replay loop: advances simulator step-by-step, calls agent at `trigger_every` interval, takes final snapshot |

**Key invariant:** Agent tools call `sim.get_history()` which only returns observations up to the current step. No lookahead is possible.

### `<package>/evaluation/`

| File | Purpose |
|------|---------|
| `metrics.py` | Computes `PerformanceMetrics` from episode snapshots and action log |
| `judge.py` | LLM-as-judge: sends original prompt + agent action trace to a judge model, returns `JudgeResult` with score (0-100) + per-violation breakdown |

### `<package>/experiment/`

| File | Purpose |
|------|---------|
| `config.py` | Pydantic models for YAML config. `ExperimentConfig.load()` handles config inheritance (`base:` field). |
| `results.py` | Serializes `ExperimentResult` to JSON, updates `results/index.json` manifest |
| `runner.py` | Wires everything together: fetch → split → simulate → metrics → judge → serialize |

### `<package>/cli.py`

| Command | Description |
|---------|-------------|
| `run --config` | Run full experiment (train + dev) |
| `run --config --split train\|dev` | Run only one split |
| `list` | Show all experiments from index.json |
| `compare name1 name2` | Side-by-side metric comparison |
| `trace name --split` | Print agent reasoning for a run |
| `fetch --config` | Pre-download and cache data without running agent |

---

## Experiment Config Format

```yaml
name: "my_experiment_v1"
description: "What this tests"
base: "configs/baseline.yaml"   # optional: inherit all fields, override selectively

data:
  source: <source_name>
  dataset: <dataset_id>
  train:
    start: "2024-01-01"
    end: "2024-03-31"
  dev:
    start: "2024-04-01"
    end: "2024-06-30"

agent:
  model: openai/gpt-5-mini       # any OpenRouter model ID
  fallback_model: null
  trigger: every_episode         # every_step | every_episode | every_day
  max_iterations: 10
  prompt:
    strategy: |
      <strategy instructions for the agent>
    constraints: |
      <constraints — these get measured by the judge>
    tone_of_voice: "Professional"

simulation:
  initial_budget: 10000.0
  fee: 0.001
  other_param: value

evaluation:
  judge_model: anthropic/claude-sonnet-4-6
  metrics:
    - metric_a
    - metric_b
    - judge_score
```

**Config inheritance:** `base: configs/baseline.yaml` deep-merges parent into child (child wins). The diff from base is stored in results for traceability.

---

## Results Format

```
results/
  index.json                          ← lightweight manifest (all experiments, key metrics)
  goals/
    001/
      h001/
        result.json                   ← aggregated across variants + runs
        variant_name/
          result.json                 ← aggregated across runs (mean ± std)
          run0/
            result.json               ← single run: metrics + judge
```

Each result JSON contains:
- `experiment`: name, timestamp, config hash, full config, diff from base
- `train` / `dev`:
  - `metrics`: domain-specific performance metrics
  - `judge_score`: 0-100
  - `judge_reasoning`: LLM explanation
  - `judge_violations`: list of specific constraint violations with step numbers
  - `reasoning`: full agent reasoning — every tool call and response per trigger

---

## How to Extend

### Add a New Data Source

1. Create `<package>/data/sources/mysource.py` implementing `get_observations()`
2. Register in `experiment/runner.py::_get_data_source()`
3. Use in YAML: `data: source: mysource`

### Add a New Agent Model

Change `agent.model` in your YAML to any [OpenRouter model ID](https://openrouter.ai/models). No code changes needed.

### Add a New Tool

1. **Schema** in `agent/core_agent.py::TOOL_SCHEMAS` — JSON schema the LLM sees
2. **Handler** in `simulation/engine.py::make_tool_handlers()` — Python function that runs

Handlers receive simulator state via closure.
