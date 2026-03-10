# Technical Runbook

> Reference for CLI commands, debugging, and environment setup. Once Claude Code has built the framework, use this to operate it.

## Running Experiments

### Single experiment
```bash
python3 -m <package> run \
  --config configs/goals/001/h001-variant_a.yaml \
  --goal 001 --hypothesis h001 --variant variant_a
```

### Multiple experiments in parallel
```bash
python3 -m <package> run-all \
  --configs-dir configs/goals/001/ \
  --workers 4 \
  --runs 6
```

### Standalone run (no goal tracking)
```bash
python3 -m <package> run --config configs/baseline.yaml
```

### Pre-fetch data (no agent)
```bash
python3 -m <package> fetch --config configs/baseline.yaml
```

## Inspecting Results

```bash
# List all experiments
python3 -m <package> list

# Compare two experiments side-by-side
python3 -m <package> compare h001-variant_a h002-variant_a

# Print agent reasoning for a specific run
python3 -m <package> trace h001-variant_a --split dev

# Read result files directly
cat results/goals/001/h001/result.json
```

## Debugging

### Agent didn't follow constraints
1. Read the run's result file or use `trace` CLI command
2. Find the trigger step where the violation occurred
3. Check what context the agent received (system prompt, state, indicators)
4. Check the judge's `judge_violations` list for specific step indices

### Experiment crashed mid-run
1. Check stderr for the error
2. Partial results may exist in `results/goals/NNN/hNNN/`
3. Re-run the same command — cached data will be reused

### Data not available
```bash
python3 -m <package> fetch --config configs/baseline.yaml
```

## Environment Setup

```bash
# Install dependencies
python3 -m pip install -r requirements.txt

# Set API keys
cp .env.example .env
# edit .env: add OPENROUTER_API_KEY=...

# Run tests
python3 -m pytest tests/ -v

# List available variants
python3 variants.py
```

## Common Patterns

### Testing a new prompt change
1. Copy an existing config: `cp configs/goals/001/h001-variant_a.yaml configs/goals/001/h002-variant_a.yaml`
2. Edit the prompt section
3. Run with `--runs 6` to average out randomness
4. Compare with previous hypothesis using `compare`

### Running all variants for a hypothesis
```bash
python3 -m <package> run-all \
  --configs-dir configs/goals/001/ \
  --pattern "h002-*.yaml" \
  --workers 4 --runs 6
```

### Checking train vs dev divergence
If train metrics are good but dev metrics are poor:
- Agent is overfitting to training prompt cues
- Try more general instructions
- Avoid including specific dates or data quirks in the prompt
