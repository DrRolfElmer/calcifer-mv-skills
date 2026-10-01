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

## 2b. State: system-wide before per-module lookup (the design default)
- **Before you add ANY new module-level state or a per-module lookup, ask: can this be ONE
  importable source?** Default answer is yes, and then it must be. Reference shape:
  `calcifer.localtime` (#153) — one time source for the whole system, no own offsets, no
  naive `now()`; guard `tools/tz_inventory.py`. → PI §4.9.
- **Preference order:** (1) importable module both sides import · (2) if it genuinely must
  stay in the assembly, a **written reason** in `tools/state_budget_baseline.json` ·
  (3) never duplicate — two sources are worse than one shared.
- **Why it is a rule and not taste:** measured 2026-08-23 — 111 of `server.py`'s 345
  module-level variables have fan-in ≥ 2, and those are exactly what each extraction wave
  must seam via `_srv()`. A shared concept in an importable module costs **zero** at a seam;
  the same concept as an assembly global costs a seam in *every* module that touches it.
- **The cost the rule does not waive:** one shared source concentrates the failure — if it
  lies, it lies to every consumer at once. Every system-wide module therefore carries the
  sentinel-default discipline (absence/unreadable = UNKNOWN, never a value) and its own guard
  that has both fired and released. → PI §3.8.
- **Guard:** `tools/state_budget.py` — a ratchet against a committed baseline; new shared
  assembly state without a written reason fails the pre-push suite.

## 2c. Stable identifiers first — device-id or MAC, never a display name
- **Address every device/host/channel by its most stable key**, in this order: platform id
  (Homey device-id, Zigbee IEEE, Shelly id, GW3000 sensor id) → MAC-bound address (a fixed IP
  counts only as a documented DHCP reservation) → display name **as fallback only**, trimmed,
  and the lookup records which key matched. Pattern: HEMS_07 rev 52 `findDev(devs, id, name)`,
  HEMS_12/13 `getDev(id, name)`. → PI §4.16.
- **Why it is a rule:** 2026-09-19 the operator renamed the Netatmo rain gauge in Homey; three
  scripts keyed on the exact name, `rain_mm_h` went null, the server's rain chain fell from the
  bucket to the piezo, four Homey rain variables froze, and the campaign script would have
  logged a frozen θ_rain for two weeks. A name is a label for humans; a rename must never
  silence a control-path source.
- **Guard (ratchet):** `tests/test_stable_identifier_ratchet.py` against
  `tools/stable_identifier_baseline.json` — 107 name lookups in `homeyscripts/*.js` measured
  2026-09-19; the count may not grow, and the baseline is tightened in the same commit a
  script migrates. Migrate when you touch a script, never as a mass edit.
- **IPs:** every hard-coded IP presupposes a DHCP reservation on MAC; the inventory is B-ID-1.

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
- **Innehåll reser aldrig i en Bash-kommandosträng.** Bash-kommandon TRUNKERAS i
  transporten vid ~8192 B (mätt 2026-08-03 över 6 078 anrop: allt ≥ 8 308 B dog, största
  lyckade 7 273 B, noll fel under 6 KB). Snittet hamnar mitt i innehållet. Med heredoc dör
  det högljutt på ett förvirrande `unexpected EOF ... matching \`''` — **utan heredoc kan en
  halv rad köras TYST**, vilket är det farliga fallet. Skriv i stället filen med **Write**
  (JSON-kanalen: ingen shell-tolkning, ingen storleksgräns, inga citatteckenproblem) och kör
  filen. Vakt: `tools/hooks/bash_cmd_size_gate.py` (PreToolUse, blockerar ≥ 7 000 B).

## 4. Delivery pipeline (in order, no skipping)
1. Run the affected test suites (+ the e2e) — green before commit.
2. Scoped commit with full message; push; verify HEAD == origin.
3. **Stage inert, promote explicit, never deploy:** `tools/stage_to_prod.py stage <files>`
   → INERT `staging\` + manifest (sha256, post_steps); `promote` (operator-approved) is the
   ONLY path to live (r4, DESIGNSPEC_STAGE_PROMOTE Fas A — an uncontrolled reboot can no
   longer activate staged code, the S16 lesson). The CAPTAIN restarts
   (`Restart-Service Calcifer`) — the agent cannot and must not. → PI §0.2.
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

- **Dev and prod may not drift apart without a reason.** `tools/check_prod_drift.py` (r5) runs
  nightly via `ci_health_check`; a RUNTIME file (server.py, calcifer/**, hems15/**, dashboard/**,
  root *.py/*.bat/*.ps1, requirements.txt) that is STALE in prod longer than the grace period
  (3 days) is **STALE UTAN SKÄL** and turns the night red — unless it has an entry in
  `tools/prod_drift_ledger.json` with a reason and a `review_by` date. The ledger is a decision,
  not a mute button: an expired entry alarms again. Operator rule 2026-09-19. → PI §0.1.

### 4a. Scheduled tasks (schtasks) — one door

- **Registration ONLY via `tools/schtasks/register_task.ps1 <xml>`** (elevated PowerShell on
  the box). A bare `schtasks /Create /XML` is banned: it is unreliable on the box (bad-syntax
  errors on valid UTF-16, credential failures on `LogonType=Password`), and it skips the
  script's gates. Operator rule 2026-09-05 after the GHI-revision task failed twice that way.
- **XML fleet standard** (what the script gates on): UTF-16LE **with BOM**, even byte count,
  CRLF, `LogonType` Password, `RunLevel` declared (`LeastPrivilege` unless the job needs
  more), `StartWhenAvailable` true, both battery blocks `false`. The legacy `tools/schtask_*.xml`
  files are UTF-8 and mostly lack `RunLevel` — convert a legacy file to the standard the day
  you touch it (as `schtask_ghi_revision.xml` was), never write a new one that way.
- **Manifest + sweep:** a new task gets a row in `tools/schtask_manifest.json`; expected
  non-zero exit codes go in `normala_exitkoder`, otherwise `schtask_sweep.py` turns red.

## 4b. CS-workbench delivery (Claude Science sandbox)
The repo mount `/mnt/c/dev/calcifer` is **read-only by design** (M&V firewall) and the
sandbox has **no git remote** (no SSH key, port 22 blocked, no GitHub credential). The §4
repo pipeline therefore runs on the OPERATOR's machine, not here. From the CS workbench:
- **Deliver, never commit.** Write outputs to `~/cs_scratch/`; the operator clones/commits/
  pushes. Repo-file fixes ship as BOTH a `.patch` and a full `*.PATCHED.py` copy so the
  committed source and any regenerated figure stay identical. Never edit committed source.
- **LaTeX builds with tectonic**, not pdflatex — both sandbox pdflatex installs are broken
  (missing `pdflatex.fmt` / perl `mktexlsr` include-path error). Use the conda `tex` env with
  `TECTONIC_CACHE_DIR=~/cs_scratch/.tectonic-cache` (multi-pass ref resolution is
  internal). The operator's own pdflatex toolchain is fine outside the sandbox.
- **Per-paper filenames are paper-suffixed from the first save.** Any deliverable sharing a
  base name across papers (`numbers_ledger`, `references`, `rigor_note`, `.zenodo.json`) →
  `numbers_ledger_paper9a.md`, etc. Bare shared names silently collapse into ONE artifact as
  successive versions (the older paper's copy gets buried), and never `version_of` an artifact
  whose earlier versions belong to a different paper — `latest_version_id` then resolves to
  the wrong paper's content.

## 5. Data & analysis discipline
- PHI never enters git-tracked docs or any cloud/analysis context
  (`hems15/**/*.csv`, `RESEARCH_LOG.md`, sleep logs). → PI §3.6 + hems15 policy.
- Frozen/byte-hashed artifacts are never "fixed" with whitespace edits. → PI §3.5/§3.6.
- Never pool experiment-window data into a normal-operation study. → PI §5 Invariant #8.
- Findings are logged with the anti-HARK discipline: pre-registered where possible,
  all-nights/all-days pooling, no cherry-picking. → PI §5.
- **Every reference you USE goes into the register.** Cited in a paper, leaned on for a
  method choice, or allowed to settle an interpretation → it belongs in
  `docs/referensregister.json` with DOI, quantified findings, searchable relevance tags,
  and — mandatory — **what it must NOT be cited for**. Query it with
  `python tools/referensregister.py --amne <topic> --full` **before** writing a section,
  so a reference reaches the next paper that needs it instead of dying in the doc where it
  first appeared (measured 2026-09-01: 96 `\bibitem` + ~157 DOIs scattered over 20 files,
  no shared source). → PI §4.10.
- **`docs/calcifer.bib` is GENERATED** (`--bibtex`), never hand-edited: a second
  hand-maintained list of the same references is two truth sources for one quantity —
  NRA-048's failure class. Zotero may capture, never be system of record (it lives outside
  git, no guard can see it). → PI §4.10, §0.3.

## Sync contract (anti-drift)
This skill cites PI by § number and copies NO prose. When a procedure changes in PI,
this skill is updated **in the same commit** (PI §0.4). `tests/test_skill_pi_sync.py`
asserts every § anchor cited here exists in the PI document — CI goes red if PI is
reorganized without this skill following.

*Impl pointers: `tools/stage_to_prod.py`, `tools/check_prod_drift.py`, `tools/mv_invariant8.py`.
Sibling skill: `invariant8-mask`. Tracked home: `docs/skills/` (`.claude/` is gitignored);
Code discovery = deliberate copy to `.claude/skills/calcifer-gold-standard/`.
Changelog: r10 (2026-10-01) — personlig hemkatalog ersatt med ~ i tectonic-exemplet (publik deposition v1.1.0). r9 (2026-09-19 kväll) — §4: dev/prod-drift utan skäl är rött (check_prod_drift r5 STALE UTAN SKÄL + tools/prod_drift_ledger.json), operatörsregel efter att fem calcifer-moduler legat efter prod i upp till 12 dygn utan larm. r8 (2026-09-19) — nytt §2c: stabila identifierare först (device-id/MAC, aldrig visningsnamn), operatörsbeslut efter att namnbytet på regnmätaren tystade HEMS_07/HEMS_13 samma dag; PI §4.16 skriven i SAMMA commit per §0.4. Vakt = spärrhaken `tests/test_stable_identifier_ratchet.py` mot baslinjen 107 (`tools/stable_identifier_baseline.json`). r7 (2026-09-01) — nytt krav i §5: varje ANVAND referens laggs in i `docs/referensregister.json` (PI §4.10 skriven i SAMMA commit per §0.4). Operatorsbeslut efter Haselow et al. (2019). Matt nulage: 96 `\bibitem` + ~157 distinkta DOI:er over 20 filer utan gemensam kalla, och paper1 bar en handskriven thebibliography ingen annan artikel kunde se. Det barande faltet ar `inte_for` — overciterande ar den vanligaste formen av felcitat och varken BibTeX eller Zotero har ett falt for en sadan grans. `docs/calcifer.bib` GENERERAS ur registret. Vakt `tests/test_referensregister.py` (14 pinnar, varje obligatoriskt falt mutationsprovat ett i taget); backfillen ar task #254. r6 (2026-08-23) — nytt §2b: systemgemensamt tillstånd före modulberoende uppslag
(operatörsbeslut; PI §4.9 skriven i SAMMA commit per §0.4). Regeln har en mätning bakom sig:
111 av server.py:s 345 modulnivåvariabler har fan-in ≥ 2 och är precis vad varje
extraktionsvåg måste sömma — en importerbar modul kostar noll vid en söm, en global i
assemblyt kostar en söm per modul. Vakt `tools/state_budget.py` + `tests/test_state_budget.py`
(task #211). Kostnadssidan inskriven: en delad källa koncentrerar felet, därav §3.8-kravet.
r5 (2026-08-03) — §3 får trunkeringsregeln: Bash-kommandon kapas vid ~8192 B,
så innehåll skrivs med Write och körs som fil. Mätt över 6 078 anrop i sessionsloggen efter
att heredocar "ätit innehåll" i flera sessioner utan att orsaken var känd; vakt i
`tools/hooks/bash_cmd_size_gate.py` + `tests/test_bash_cmd_size_gate.py` (task #76).
r4 (2026-07-21) — §4 step 3 rewritten for the stage/promote split
(stage_to_prod r4, DESIGNSPEC_STAGE_PROMOTE Fas A; S16 remediation).
r3 (2026-07-19) — §4b CS-workbench delivery conventions (RO-repo patch delivery
to ~/cs_scratch as .patch + *.PATCHED copy, tectonic LaTeX builds, paper-suffixed artifact
filenames), harvested from the Claude Science publication-workbench sessions; these are
CS-sandbox operational mechanics (not PI-derived, so no new PI § anchor).
r2 (2026-07-12) — §4.7 CI-minute-budget discipline ([skip ci]/[full-ci]/smoke
tiering), harvested from the Actions billing-block incident alongside ci.yml r4.
r1 (2026-07-10) — first distillation of PI §0/§0.1/§0.2/§0.3/§3.6/§4/§5 into an
operational skill, per operator decision ("Hur säkerställer vi att detta fångas i en skill?").*
