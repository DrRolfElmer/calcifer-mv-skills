---
name: calcifer-gold-standard
description: >
  The Calcifer development gold standard — the operational distillation of
  PROJECT_INSTRUCTIONS §0/§4/§5. Use whenever writing, committing, deploying,
  reviewing or analysing ANYTHING in the Calcifer repo — every code change, doc
  change, HomeyScript push, prod staging or M&V analysis — even if the user never
  mentions guidelines. This is the fickmanual; PI is the lagbok (system of record).
---

# Skill: calcifer-gold-standard

**One-line:** Run a tight ship — no orphans, no ghosts, no sloppy parts; verify, never
trust; the operator restarts, nothing more. Every rule below is a POINTER into
`docs/PROJECT_INSTRUCTIONS_rev25.md` (PI) — read the cited § when judgment is needed.
PI is the single source of truth; this skill never copies PI prose, only operationalizes it.

## 1. Pre-flight (before touching anything)
- Read the PI § that owns the area you are changing (component table: PI §1; invariants: PI §3).
- Analysing time-series data? Load it via the `invariant8-mask` skill / `tools.mv_invariant8`
  (never bare `read_csv`) — experiment windows + freeze-traps. → PI §5 Invariant #8.
- Control-path, MPC-objective or IPMVP-baseline adjacent? STOP and check the M&V firewall
  first (observation-only / shadow-first / deploy-dark / kill-switch). → PI §3.6, NRA-021.

## 2. Change discipline
- **One concern per rev.** Split unrelated fixes into separate revs/commits.
- **Bump the version token** (`SERVER_REV` for server.py, `__rev__` for tools/tests/scripts)
  **+ inline changelog line** — the pre-commit gate enforces presence, YOU ensure the
  changelog actually explains the why. → PI §4 (VERSIONING_CONVENTION).
- **Scoped `git add <paths>` — NEVER `git add .`** (PHI + frozen artifacts live in-tree).
- Patch programmatically (targeted edits), don't regenerate whole files. → PI §4.1.
- New/removed files: run the orphan process — nothing declared-but-never-written,
  staged-but-never-deployed, or referenced-but-deleted. → PI §0.1.

## 3. Verification discipline (the anti-trust rules)
- **A screen is not a finding; a negative match is not absence.** Grep output, empty diffs
  and "(no output)" must be POSITIVELY explained before you act on them. → PI §0.
- **String replaces silently no-op.** After every scripted edit: assert-verify the change
  landed in the file (read it back), never report "updated" from the script's own print.
- **Red/erroring/skipped tests are root-caused, NEVER bypassed** (`--skip-tests` is how the
  2026-06-15 deploy-guard silently died for 36 h). → PI §5 red-test anti-pattern.
- Pre-commit hooks may normalize line endings and ABORT the first commit attempt: after any
  commit, verify with `git log`/`git status` that it actually landed — and put the FULL
  message on the retry, not "2nd pass".

## 4. Delivery pipeline (in order, no skipping)
1. Run the affected test suites (+ the e2e) — green before commit.
2. Scoped commit with full message; push; verify HEAD == origin.
3. **Stage, never deploy:** `tools/stage_to_prod.py <files>` (backup + sha-verify).
   The CAPTAIN restarts (`Restart-Service Calcifer`) — the agent cannot and must not. → PI §0.2.
4. After the operator's restart: health-check `/health` (server_rev, composite_hash,
   warnings, the touched subsystem's block) before declaring success.
5. **Operator-facing artifacts (runbooks, field sheets, evals) → committed to `docs/`
   the SAME session.** Chat is notification; repo is home. → PI §0.3.
6. Update the paper trail the same session: PI component row + chronology, master backlog,
   research log if a finding, memory if a durable lesson.
7. **CI-minute budget** (`.github/workflows/ci.yml` r4 — 2 000 free min/month; the
   Windows runner bills 2×): the LOCAL pre-push pytest gate is the quality gate;
   cloud CI is a backstop. Per-push cloud = ubuntu smoke only. Push discipline:
   batch WIP pushes with `[skip ci]` in the commit message (suppresses even smoke);
   plain push at milestones; add `[full-ci]` for risky/env-sensitive changes (they
   get the Windows full suite immediately instead of the 02:45 UTC nightly).
   Doc-only pushes are already free (paths-ignore). Never spend 2× Windows minutes
   re-verifying what the local gate just verified on the same OS.

## 5. Data & analysis discipline
- PHI never enters git-tracked docs or any cloud/analysis context
  (`hems15/**/*.csv`, `RESEARCH_LOG.md`, sleep logs). → PI §3.6 + hems15 policy.
- Frozen/byte-hashed artifacts are never "fixed" with whitespace edits. → PI §3.5/§3.6.
- Never pool experiment-window data into a normal-operation study. → PI §5 Invariant #8.
- Findings are logged with the anti-HARK discipline: pre-registered where possible,
  all-nights/all-days pooling, no cherry-picking. → PI §5.

## Sync contract (anti-drift)
This skill cites PI by § number and copies NO prose. When a procedure changes in PI,
this skill is updated **in the same commit** (PI §0.4). `tests/test_skill_pi_sync.py`
asserts every § anchor cited here exists in the PI document — CI goes red if PI is
reorganized without this skill following.

*Impl pointers: `tools/stage_to_prod.py`, `tools/check_prod_drift.py`, `tools/mv_invariant8.py`.
Sibling skill: `invariant8-mask`. Tracked home: `docs/skills/` (`.claude/` is gitignored);
Code discovery = deliberate copy to `.claude/skills/calcifer-gold-standard/`.
Changelog: r2 (2026-07-12) — §4.7 CI-minute-budget discipline ([skip ci]/[full-ci]/smoke
tiering), harvested from the Actions billing-block incident alongside ci.yml r4.
r1 (2026-07-10) — first distillation of PI §0/§0.1/§0.2/§0.3/§3.6/§4/§5 into an
operational skill, per operator decision ("Hur säkerställer vi att detta fångas i en skill?").*
