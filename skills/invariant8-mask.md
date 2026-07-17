---
name: invariant8-mask
description: >
  Load any Calcifer M&V / time-series CSV (savings, phase-load, DHW, thermal-ID,
  effekt-peak, sleep) with automatic PI Invariant #8 experiment-window flagging and
  the /mpc/solve freeze-trap. Use this INSTEAD of a bare pandas.read_csv at the start
  of EVERY Calcifer analysis that touches logged time-series data — even if the user
  never mentions experiments — so experiment-contaminated or silently-frozen rows
  never enter a normal-operation study.
---

# Skill: invariant8-mask

**One-line:** Load any Calcifer M&V CSV with automatic Invariant #8 experiment-window
flagging — never analyse experiment-contaminated (or silently frozen) data by accident.

## When to use
Use this **instead of a bare `pd.read_csv()`** at the start of *every* Calcifer analysis
that touches time-series M&V data (savings, phase load, DHW, thermal-ID, sleep, effekt-peak).
PI Invariant #8 requires each analysis to check its window against active experiments; this
skill makes that automatic and reproducible.

## What it does
1. **Discovers experiment windows** — Solar Float runs auto-detected from
   `data/solarfloat/<run_id>/meta.json` (`t_float_ms`→`t_end_ms`), plus a maintainable
   `tools/experiment_windows.json` registry for experiments with no run dir (Philips OFF,
   HEMS_12 Step-Down/PhaseDetect, HEMS_17 PhaseReconcile, …).
2. **Annotates the frame** — adds `in_experiment_window` (bool) and `experiment_tag`
   (which window(s) each row fell in).
3. **Guards the freeze trap** — if the file is in the §1b FROZEN list *and* its span
   overlaps an MPC-suspending window, it warns loudly: those rows are **stale** (mtime
   stuck at window start), so excluding rows won't save you — use a still-logging source.

## Usage
```python
from tools.mv_invariant8 import load_mv_csv, exclude_experiments

df = load_mv_csv("phase_actuals_15min_log.csv")   # auto-detects ts col + windows
clean = exclude_experiments(df)                    # drop contaminated rows for normal-op studies
```
CLI pre-flight:
```
python -m tools.mv_invariant8 --csv daily_savings_log.csv
```

## Maintaining the registry
Add a window to `tools/experiment_windows.json` whenever you run a non-Float experiment.
Set `suspends_mpc: true` for HEMS_12/13/17 (freezes the `/mpc/solve` CSVs); `false` for
observational experiments like Philips OFF (regime/exposure contamination only).

## Notes / limits
- Freeze list mirrors `docs/SOLARFLOAT_ANALYSIS_PLAN.md §1b` — keep the two in sync.
- Timestamps: ISO-8601 (tz-aware or date-only) and epoch-ms are auto-handled; pass
  `ts_col=` to override detection.
- **PHI:** the sleep CSVs are PHI — this skill only reads timestamps/flags, but do not carry
  their rows into a Claude Science analysis *context* (see `docs/CLAUDE_SCIENCE_EVAL.md §4`).
- Packaging: this is the plain-Python form of the skill. If/when the Claude Science app's
  skill-manifest format is confirmed, wrap this module per that spec (the logic is unchanged).
- **Tracked home (repo audit 2026-07-09):** `docs/skills/invariant8-mask/SKILL.md` is the
  git-tracked canonical copy — `.claude/` is **gitignored** in this repo, so the
  `.claude/skills/` convention from the CS session would silently untrack it. For live
  Claude Code discovery, copy (deliberately) to `.claude/skills/invariant8-mask/SKILL.md`;
  frontmatter verified against code.claude.com/docs/en/skills 2026-07-09 (name +
  description valid; `paths:` glob-activation is an available future option).

*Impl: `tools/mv_invariant8.py` (r2 — repo audit added the post-§1b solve-driven frozen stems: perzone_k_shadow / deferrable_log / effekt_shadow). Tests: `tests/test_mv_invariant8.py`.
Cross-ref: PI Invariant #8, `[[simulation-modelling-routes]]`, `docs/CLAUDE_SCIENCE_EVAL.md §8`.
Changelog: r1 (2026-07-09) — YAML frontmatter added (name + triggering description); body unchanged from operator draft.*
