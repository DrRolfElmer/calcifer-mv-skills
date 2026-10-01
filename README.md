# Supplementary material — machine-readable M&V agent skills

Supplementary deposit for the paper *"Encoding measurement-and-verification discipline as
machine-readable skills so an AI agent enforces it"* (Rolf Elmér, preprint,
2026).

## What this is
The four **agent skills** that are the paper's central artifact — short, structured, machine-readable
methodology documents that an AI analysis agent loads automatically when a task matches their
triggering description. Together they encode the mechanical half of the research programme's
measurement-and-verification (M&V) discipline so the agent enforces it *by construction*.

| Skill | Lines | Role |
|---|---|---|
| `calcifer-gold-standard` | 199 | operational / dev standard |
| `calcifer-research-standard` | 71 | epistemic ranking, non-circularity, corrections, honest limits |
| `invariant8-mask` | 68 | data loader flagging experiment-contaminated and frozen-logger rows |
| `dhw-cycle-analysis` | 77 | domain signal-semantics pitfalls |

## Versions
- **1.1.0 (October 2026)** — the skills as of 1 October 2026, the repository state behind the line
  counts in the paper's §2. Gold-standard grew from 86 to 199 lines and dhw-cycle-analysis from 57 to
  77 lines since 1.0.0; research-standard and invariant8-mask are unchanged.
- **1.0.0 (July 2026)** — the initial deposit.

## Scope of this deposit
- **Included:** the four `SKILL.md` documents (methodology only). They are self-contained and
  reusable — the *structure* (loadable rules + a rigor gate) transfers; the specific rules are this
  programme's and would be re-authored elsewhere.
- **Not included:** the home-energy controller source code (on a staged 2028 release path) and any
  personal/household data. The skills-and-gate pattern is reproducible without either.

## How this maps to the paper
The paper argues that encoding M&V discipline as loadable skills, backed by executable guards and a
fresh-context rigor gate, makes hard-won conventions the *default* behaviour of an AI analysis agent
and makes them *checkable*. These four files are the encoded discipline. The logged worked examples
in §3 of the paper, including the post-freeze evidence in §3.6, are backed by de-identified project
artifacts **available on request**.

## Citation
Cite the paper as the primary reference; cite this deposit by its Zenodo concept DOI
(10.5281/zenodo.21420254, all versions) or by the version DOI of 1.1.0.

## Reader's note on internal cross-references
These skills were authored for internal use and cross-reference a private project-instructions
document (cited as `PI §…`), internal risk-analysis notes (`NRA-…`), tool source files
(`tools/…`), and the project research log (`RESEARCH_LOG…`). Those artifacts are part of the
controller codebase (on a staged 2028 release path) and are **not** included here. The skills are
published as *methodology* and are readable and reusable without them — the pointers indicate where,
in the original programme, each rule is grounded; they are intentional provenance, not broken links.
Device and platform names that appear in examples (a home-automation hub, sensors, meters) identify
the class of equipment a rule was learned on, not a recommendation.

## License
Creative Commons Attribution 4.0 International (CC-BY-4.0) — see `LICENSE`.

## Assembly (single source of truth)
The canonical skills live in the project repository under `docs/skills/<name>/SKILL.md`. Build the
upload bundle from the canonical files at deposit time (see `MANIFEST.md`) so the deposit never
drifts from the source.
