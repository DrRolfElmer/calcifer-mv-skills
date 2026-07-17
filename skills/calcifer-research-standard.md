---
name: calcifer-research-standard
description: >
  The Calcifer research methodology standard — use for EVERY analysis, figure,
  manuscript or scientific claim on Calcifer data (Claude Science kernels, Code
  analysis sessions, agent-run explorations), even if the user never mentions
  methodology. Companion to calcifer-gold-standard (dev) and invariant8-mask
  (data loading): this one governs how CONCLUSIONS are formed, hedged, corrected
  and published.
---

# Skill: calcifer-research-standard

**One-line:** Measurement beats manual beats model beats memory; hypotheses carry named
resolvers; null findings and corrections are published, not buried. Pointers into PI §5
and the research log's worked examples — never copied prose.

## 1. The epistemic ranking (when sources disagree)
**Measurement > manufacturer documentation > model > memory.** Worked examples
(RESEARCH_LOG 2026-07-09/10): the HP heats solo to 68 °C though the manual says
THP_MAX=55; panel symbols lit ≠ element drawing; MMI params turned out ECO-only while
the unit runs [A]. → When a spec contradicts a clean measurement, the measurement wins
and the contradiction is ITSELF a logged finding. "A screen is not a finding; a negative
match is not absence" (PI §0) applies to data: an empty grep/absent rows must be
positively explained (frozen channel? freeze-trap? filter bug?) before use.

## 2. Non-circularity (the cardinal rule for model validation)
Never validate a model against its own output. B1's rise IS cop_model — its parity value
comes only from INDEPENDENT truth (MMI anchors, the unit's sensor, measured Wh).
COP back-outs: energy balance from anchors + measured electrical energy
(`tools/dhw_model_calibrate.estimate_cop_from_segment`); BSH segments verify C
COP-free; standby gives UA. Two models agreeing proves nothing (legacy matched the
sensor by two errors cancelling — twice).

## 3. Hypothesis discipline: rank + NAME THE RESOLVER
An open question gets (a) ranked hypotheses, (b) the datum that will discriminate them,
(c) WHEN that datum arrives. Worked example: the B1 long-cycle overshoot — draws vs COP-a1
vs losses; resolver = the next Grohe daily row; "unresolvable tonight" stated instead of
concluded. Never publish the leading hypothesis as the finding.

## 4. Null findings and honest limits are DELIVERABLES
A null with a diagnosed cause is a finding (B.3: σ(wind)=0 → frozen logger channel →
ops fix + the blind abort-guard discovery). Every analysis/manuscript carries an
honest-limits section (N=1, single week, C-normalized θ convolution, season transfer
pending — see PUBLICATION_ANGLES "Honest generalization limits"). Exploratory work is
LABELLED exploratory (Tier-B) and never silently promoted to confirmatory.

## 5. Corrections are public and append-only
Wrong numbers get a CORRECTION entry in the research log (never edit the original —
the 26-min/120-min cycle correction, the "4/7-of-11"→"6/5-of-11" H4 correction).
Evidence CSVs are append-only: correct at ANALYSIS time (the perzone F4 rule), never
rewrite the log.

## 6. Anti-HARK / pooling (PI §5)
Pre-register where possible; pool ALL nights/days in a window — no cherry-picking;
check every window against experiment regimes via the `invariant8-mask` skill (its
freeze-trap output belongs in the methods note). Anything post-hoc is labelled post-hoc.

## 7. Reproducibility & provenance
Every number traces to a file+timestamp (no memory numbers). Figures are produced by
committed code paths (the tools/, not ad-hoc notebook cells) — in Claude Science every
figure artifact carries code+env+narrative, and the reviewer agent is run on figures
BEFORE manuscript text cites them. Reviewer-agent limitation: it is the same model
checking itself — it catches the mechanical half (untraceable numbers, figure/code
mismatch), it does NOT replace this skill's judgment rules.

*Sources: PI §0 + §5 (anti-HARK, Invariant #8) · docs/RESEARCH_LOG_ENERGY (worked
examples cited above) · docs/PUBLICATION_ANGLES_2026-07.md · docs/CLAUDE_SCIENCE_EVAL.md
§11 · sibling skills: calcifer-gold-standard, invariant8-mask, dhw-cycle-analysis.
Changelog: r1 (2026-07-10) — harvested under the skill-harvest standing rule (operator:
"best practice i en skill"), distilling PI §5 + the 07-09/10 worked examples.*
