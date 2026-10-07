# minimax Quota Gate Reference

## Background

minimax Token Plan (old-user plan, pre-2026-03-22) has:
- 5-hour rolling quota: ~40M tokens per window
- 5 resets per day (0, 5, 10, 15, 20 UTC+8) → daily ~200M
- Monthly cap: ~60B tokens

If the 5-hour quota is exceeded, the API **automatically deducts from the resource pack (资源包)**. Most users want to avoid this.

## Quota Gate Rule

Before any `uniagent:` forward, check the quota. If ≤ 10% remaining, do not forward.

```bash
mmx quota --output json
```

Parse `model_remains[0].current_interval_remaining_percent`:
- `> 10` → safe to forward
- `≤ 10` → refuse, tell user

## Thresholds

- `> 10%` → forward
- `≤ 10%` → refuse

(Originally considered 30/10 thresholds, but user prefers simple binary.)

## Cost Note

A single `mmx quota` call consumes only a few K tokens (well below 1% granularity = 400K tokens), so it's fine to run on each `uniagent:` command. Do NOT run quota gate before every tool call — that would be wasteful.