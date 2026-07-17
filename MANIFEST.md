# Deposit manifest — build the Zenodo bundle from canonical sources

The upload bundle is assembled at deposit time from the canonical skill files so the deposit never
drifts from the repository. Files to include in the Zenodo upload:

| In bundle as | Canonical source (repo) | Lines |
|---|---|---|
| `skills/calcifer-gold-standard.md` | `docs/skills/calcifer-gold-standard/SKILL.md` | 86 |
| `skills/calcifer-research-standard.md` | `docs/skills/calcifer-research-standard/SKILL.md` | 71 |
| `skills/invariant8-mask.md` | `docs/skills/invariant8-mask/SKILL.md` | 68 |
| `skills/dhw-cycle-analysis.md` | `docs/skills/dhw-cycle-analysis/SKILL.md` | 57 |
| `README.md` | `docs/drafts/paper9b/supplementary/README.md` | — |
| `LICENSE` | `docs/drafts/paper9b/supplementary/LICENSE` | — |
| `.zenodo.json` (metadata) | `docs/drafts/paper9b/supplementary/.zenodo.json` | — |

## Assembly (run from the repo root)
```
mkdir -p build/ccai_supplementary/skills
cp docs/skills/calcifer-gold-standard/SKILL.md   build/ccai_supplementary/skills/calcifer-gold-standard.md
cp docs/skills/calcifer-research-standard/SKILL.md build/ccai_supplementary/skills/calcifer-research-standard.md
cp docs/skills/invariant8-mask/SKILL.md          build/ccai_supplementary/skills/invariant8-mask.md
cp docs/skills/dhw-cycle-analysis/SKILL.md       build/ccai_supplementary/skills/dhw-cycle-analysis.md
cp docs/drafts/paper9b/supplementary/README.md   build/ccai_supplementary/README.md
cp docs/drafts/paper9b/supplementary/LICENSE     build/ccai_supplementary/LICENSE
cp docs/drafts/paper9b/supplementary/.zenodo.json build/ccai_supplementary/.zenodo.json
( cd build && zip -r ccai_supplementary_v1.zip ccai_supplementary )
```

## Pre-upload checklist
- [ ] Each `SKILL.md` re-read for any project-internal path, host, or identifier that should not be
      public (the skills are methodology; verify no secrets/PHI/host addresses leak).
- [ ] `.zenodo.json` `related_identifiers` updated with the paper's arXiv/venue id once assigned.
- [ ] Author ORCID/affiliation confirmed; add co-authors if any.
- [ ] License confirmed (CC-BY-4.0) and consistent across README, LICENSE, and `.zenodo.json`.
