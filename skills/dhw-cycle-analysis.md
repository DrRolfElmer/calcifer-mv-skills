---
name: dhw-cycle-analysis
description: >
  Analyse Calcifer DHW/EKHHP charge cycles, tank-temperature signals or COP — use
  whenever working with dhw_power_log, dhw_tsim_audit, MMI readings or the B1
  estimator, even if the user doesn't mention pitfalls. Encodes the 2026-07-10
  hard-won signal semantics: mid-cycle reads lie, the proxy holds stale transients,
  short vs long cycles behave differently, and which of the three temperature
  signals to trust when.
---

# Skill: dhw-cycle-analysis

**One-line:** Three temperature signals, none of them simply "the truth" — know which
lies when, run the tool instead of eyeballing, and never conclude from a running cycle.

## The tool (always start here)
```
.venv/Scripts/python.exe tools/dhw_b1_parity_analyze.py [--since YYYY-MM-DD] [--out x.json]
```
Cycle census (HP-only check), per-cycle parity (sensor-lead, proxy-hold), standby-UA
fit, autonomous-restart detector. Data: `dhw_power_log.csv` (60 s during charge) +
`dhw_tsim_audit.csv` parity columns. Experiment windows? Load via `invariant8-mask` first.

## Signal semantics (the 2026-07-10 lessons — each cost us a wrong conclusion)
| Signal | Trust it for | It LIES when |
|---|---|---|
| **MMI display** (manual read) | Absolute truth at THAT minute **at the top sensor** | **±1–1.5 K minutes-scale wiggles with ZERO power input** (layer dynamics/draws — 2026-07-12; cross-check the power log before calling a rise a charge). Sparse; anchor recipe below |
| **r21 proxy** (audit col 4) | Trends, standby decay | **Post-charge: HOLDS the stratification transient** (+3 K stale for ~1 h while the MMI relaxes) |
| **B1 estimator** (t_sim_b1_c) | Energy-weighted lump truth; SHORT cycles ±0.5 K | **LONG cycles: overshoots** (no draw term v1 + warm-T_out COP a1 high — +9 K on the 07-10 deep precharge); no target clamp BY DESIGN |
| **legacy t_sim** | Nothing directly | Models the COMMAND not the measurement (jumps pre-compressor); clamps at target → "right for wrong reasons" on long cycles |

## Hard rules
1. **Never conclude from a RUNNING cycle** — the "26-min cycle 2" error (2026-07-10): it
   was 120 min; the read was mid-cycle. Check compressor_w has been <300 W for ≥5 min
   (or the audit mode returned to IDLE) before calling a cycle complete.
2. **Panel symbols ≠ power** — trust the Shelly (heater glyph lit ≠ BSH drawing).
3. **Short vs long cycles are different regimes:** ≤30 min = stratification-dominated
   (sensor leads the lump by ~3 K, relaxes in ~40 min); ≥1 h = draws + COP dominate the
   B1 divergence. Report them separately.
4. **COP back-outs must be non-circular:** anchors (MMI) + measured Wh only — never from
   B1's own trajectory (B1 IS the model). Method: `tools/dhw_model_calibrate.py`
   `estimate_cop_from_segment` (energy balance; BSH verifies C, standby gives UA).
5. **Draws are unmeasured intraday** — the Grohe row is DAILY (~00:xx). A long-cycle
   energy gap is UNRESOLVABLE until the next day's row; say so instead of concluding.
6. **HP-only census continues:** every cycle, check BSH max <300 W ([4-03]=2 predicts
   autonomous = HP-only, ~1.2 kW; any BSH engagement is a FINDING — log it).

## Tvåregimsregeln + transientdomänen (r3, 2026-07-26 — S3-kampanjen + QA)
- **Två driftregimer, blanda aldrig i en pool:** SG-kommenderad (ch1 ≈ 5 W; band
  4,05 ± 0,24) vs autonom helsekvens (aux 500–714 W i ~114 min varav ~73 FÖRE
  kompressorn; dörr-till-dörr 2,12–2,28 = 1,9× dyrare). Autonom trigger ≈ 41°.
- **Transientens domän är HÖGA MÅL, inte laddningsdjup:** 0,0 K × 3 (grunda + djup
  44→56) mot +1,0 K vid 62°-toppen. Vid-stopp-avläsningar ≤57° är pålitliga;
  55–62-bandet kräver satta ankare. ("Djup ⇒ transient"-förutsägelsen FÖLL 26/7.)
- **Kantkorrektionen är obligatorisk för korta cykler:** loggern vilar på 5-min-kadens
  ⇒ varje start döljer 2–4 min drifttid. `s3_shortcycle_analyze` r4+ löser luckan ur
  enhetens register (relativt trogna ~0,1 %; absolut ~13,5× låga — ALDRIG som energi).
- **T_start efter dragning = 15 min väntan + 0,5 K-regeln** (samma fälla som T_end,
  andra änden — punkt A-kontamineringen). OBS: en dragning kan SPÄNNA över fotot
  (grohe-flödet avgör, sleepqa/s3qa P2b).

## MMI anchor recipe (operator fieldwork, maximizes anchor value)
Read **before** the cycle · **right after** compressor stop (transient top) · **+40 min**
(mixed truth). Photo per read; timestamps from the display — trustworthy because
**the operator resets the MMI clock after every power outage** (standing practice,
#115: the clock has no outage backup). Only the window between an outage and the
reset is suspect; there, use the phone photo's metadata.

*Sources: DESIGNSPEC_DHW_EKHHP_MPC_MODEL.md §E/§8 · RESEARCH_LOG_ENERGY 2026-07-09/10
entries · tools/dhw_b1_parity_analyze.py r1 · tools/dhw_model_calibrate.py r2 · PI §0
(a screen is not a finding). Changelog: r5 (2026-09-07) — correction: operator resets
the clock after every outage, historical display times clean; suspect window = outage→reset.
r4 (2026-09-07) — MMI clock resets on power
outages (#115); anchor timestamps from phone-photo metadata. r3 (2026-07-26) — tvåregimsregeln, transientdomänen, kantkorrektionen, T_start-väntan (S3-kampanjen + s3qa-QA:n). r2 (2026-07-12) — MMI point-sensor wiggle rule (66→67.5 with 0 W measured,
L2 at household baseline — layer dynamics, not heating). r1 (2026-07-10) — first harvest under the
skill-harvest standing rule.*
