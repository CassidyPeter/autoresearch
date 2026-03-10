# Compute Guide

## Priority

Always try the primary resource first. Fall back to secondary if unavailable.

| Priority | Resource | Notes |
|----------|----------|-------|
| 1 | Primary (e.g. H200 GPU server) | Fastest; prefer for large experiments |
| 2 | Fallback (e.g. A100, local) | Use direct Python — no Slurm or scheduler |

## Connecting

```bash
# Primary
ssh user@primary-host

# Fallback
ssh user@fallback-host
```

## Running on Primary

```bash
# Activate environment
source /path/to/env/bin/activate

# Run experiment
python3 -m <package> run --config configs/goals/001/h001-variant_a.yaml \
  --goal 001 --hypothesis h001 --variant variant_a
```

## Running on Fallback

```bash
# Direct Python (no scheduler)
python3 -m <package> run-all \
  --configs-dir configs/goals/001/ \
  --workers 2 --runs 6
```

## Environment Variables

Copy `.env.example` to `.env` and fill in:

```
OPENROUTER_API_KEY=sk-or-...
DATA_SOURCE_API_KEY=...     # if your data source requires authentication
```

## Resource Estimates

| Experiment size | Workers | Time (primary) | Time (fallback) |
|----------------|---------|----------------|-----------------|
| 1 variant, 6 runs | 1 | ~5 min | ~10 min |
| 5 variants, 6 runs | 4 | ~8 min | ~20 min |
| Full hypothesis (all variants) | 4 | ~15 min | ~40 min |

## Troubleshooting

- **Connection refused:** Check if the host is up. Try fallback.
- **OOM / killed:** Reduce `--workers`. Use `--runs 3` first to verify, then full 6.
- **Slow data fetch:** Run `fetch` command first to warm cache, then run agents.
