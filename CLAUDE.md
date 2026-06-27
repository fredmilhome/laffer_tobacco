# CLAUDE.MD -- Cigarette Tax Revenue & Consumption Responses in Brazil

<!-- Working research project. Primary analysis language: R. Python is used only by the
     inherited Claude Code template hooks (.claude/hooks/*.py). Slide/Beamer/Quarto
     infrastructure is kept DORMANT for an eventual presentation/paper write-up.
     Full design context: project_context.md. Keep this file lean — loaded every session. -->

**Project:** Cigarette Tax Revenue and Consumption Responses in Brazil
**Branch:** main

Simulating cigarette tax-revenue and consumption responses to (i) the D12922/2026 step changes
(specific IPI R$2.25→R$3.50; minimum price R$6.50→R$7.50) and (ii) Imposto Seletivo (IS) reform
scenarios, on a calibrated 2019 baseline using region × price-category elasticities, the Brazilian
tax-stack formula, and a normalized-CES Laffer framework. See [project_context.md](project_context.md).

---

## Core Principles

- **Plan first** -- enter plan mode before non-trivial tasks; save plans to `quality_reports/plans/`
- **Verify after** -- run the pipeline / re-render outputs and confirm numbers at the end of every task
- **Single source of truth** -- `scripts/R/_outputs/*.rds` are authoritative; every table, figure, and
  paper number derives from them. **All parameters live in `scripts/R/00_config.R` — never hardcode**
  elasticities, rates, or the calibration target in analysis scripts.
- **Calibration is sacred** -- the model must reproduce the 2019 baseline `R_0 = R$17.75 bn`; the
  illicit-share adjustment is the only calibration knob (see `.claude/rules/project-conventions.md`)
- **Quality gates** -- nothing ships below 80/100
- **[LEARN] tags** -- when corrected, save `[LEARN:category] wrong → right` to [MEMORY.md](MEMORY.md)

Cross-session context lives in [MEMORY.md](MEMORY.md); past plans, specs, and session logs are in [quality_reports/](quality_reports/).

---

## Folder Structure

```
laffer_tobacco/
├── CLAUDE.MD                    # This file
├── project_context.md          # Research design, literature, data, pipeline spec
├── .claude/                     # Rules, skills, agents, hooks
├── Bibliography_base.bib        # Centralized bibliography (+ bib.bib working refs)
├── data/
│   ├── raw/                     # VIGITEL, PNS, PNAD, IBGE, RFB, CONFAZ (NOT committed — see data/README.md)
│   └── processed/               # state_icms_rates.csv, baseline_calibrated.csv
├── scripts/R/                   # Analysis pipeline: 00_config → 07_results (authoritative outputs)
│   └── _outputs/                # Generated .rds (gitignored)
├── regulations/                 # HTML of key decrees (D12922, D12127, D7555, PLP68, EC132)
├── papers/                      # .md transcriptions of key papers
├── results/                     # Output tables/figures by scenario
├── quality_reports/             # Plans, specs, session logs, audits, diagnoses
├── explorations/                # Research sandbox (see rules)
├── templates/                   # Session log, spec, quality-report templates
└── master_supporting_docs/      # Source papers and existing slides

# DORMANT (presentation/paper phase — kept from the workflow template, not active yet):
#   Slides/  Quarto/  Figures/  Preambles/  docs/  guide/
```

---

## Commands

```bash
# Run the full R pipeline (00_config sourced first, then 01→07)
Rscript scripts/R/00_run_all.R

# Run a single stage
Rscript scripts/R/03_analyze.R

# Quality score (advisory gate)
python scripts/quality_score.py <file>

# DORMANT — presentation phase only:
#   ./scripts/sync_to_docs.sh <Deck>      # deploy Quarto to GitHub Pages
#   ./scripts/check-palette-sync.sh        # LaTeX ↔ SCSS palette contract
```

---

## Quality Thresholds (advisory)

| Score | Checkpoint | Meaning |
|-------|------|---------|
| 80 | Commit | Good enough to save |
| 90 | PR | Ready for deployment |
| 95 | Excellence | Aspirational |

Enforced by `/commit` (halts + asks for override) **and** — once you run `./scripts/install-hooks.sh` —
by a git pre-commit hook (`.githooks/pre-commit`). Bypass sparingly with `SKIP_QUALITY_GATE=1` or `--no-verify`.

---

## Skills Quick Reference

Full index in [README.md](README.md#skills-claudeskills). Most-used here, by workflow:

- **Data / reproducibility (primary):** `/data-analysis` `/audit-reproducibility` `/diagnose`
  `/simulation-study` `/replication-package` `/capture-environment` `/disclosure-check` `/did-event-study`
- **Research / writing:** `/interview-me` `/lit-review` `/research-ideation` `/preregister`
  `/review-paper` (`--peer`) `/verify-claims` `/proofread` `/respond-to-referees`
- **Meta / workflow:** `/commit` `/learn` `/checkpoint` `/context-status` `/deep-audit`
- **Dormant (presentation phase):** `/create-lecture` `/compile-latex` `/deploy` `/qa-quarto`
  `/slide-excellence` `/teach-from-paper` `/extract-tikz`

---

## Project-Specific Conventions

Binding invariants (tax formula, calibration target, exact-CES default, elasticity provenance,
config-not-hardcoded, illicit-share calibration) live in
[`.claude/rules/project-conventions.md`](.claude/rules/project-conventions.md) and apply to
`scripts/R/**` and `data/**`. Quick numbers reference: [project_context.md §9](project_context.md).

---

## Current Project State

| Phase | Scope | Output | Status |
| --- | --- | --- | --- |
| 0: Setup | Workflow config, dirs, conventions | This config | In progress |
| 1: Direct estimates | Hardcoded elasticities (Divino 2024); calibrate 2019 baseline; simulate D12922 + IS grid | Revenue/consumption tables by scenario | Not started |
| 2: Own elasticities | Two-part model (probit + log-log OLS) on PNS 2013+2019; region × PC | Estimated elasticities → pipeline | Not started |
| 3: Cross-elasticity stress test | Re-run Phase 1 under `ε^{I,L} ∈ {0, 0.3, 0.6}` | Laffer-peak sensitivity | Not started |

<!-- Beamer environments / Quarto CSS tables: DORMANT — populate when the presentation deck starts. -->
