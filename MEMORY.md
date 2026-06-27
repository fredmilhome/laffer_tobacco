# Project Memory — Tobacco Tax Laffer

Cross-session facts and corrections for the cigarette tax-revenue / consumption simulation.
When a mistake is corrected, append a `[LEARN:category] wrong → right` entry under the log.

> Template-build history (how the workflow template itself was developed) was moved to
> `quality_reports/template_memory_archive.md` on 2026-06-27 — it is not relevant to running
> this study. Keep this file < 200 lines.

---

## Project Conventions (binding — see `.claude/rules/project-conventions.md`)

- **Calibration target `R_0 = R$17.75 bn` (2019).** Only knob is the illicit (PC1) share. Re-run
  calibration whenever the price floor changes (R$7.50 reclassifies PC2→PC1).
- **Exact CES is the default** demand form (Phase 1+); linearized only to replicate Divino 2021.
- **Config-not-hardcoded:** all parameters live in `scripts/R/00_config.R`.
- **Laffer peak exists only if `|ε_total| > 1`** — depends on including the participation margin.
- `ε^{I,L} ≈ 0` (Divino 2025 JAE) is the result being **stress-tested** ({0, 0.3, 0.6}), not inherited.
- Primary language **R**; Python only for `.claude/hooks/*.py`. Slide infra is dormant.
- SSOT: `scripts/R/_outputs/*.rds` are authoritative; never hand-edit derived tables/figures.

## Workflow Patterns (carried from template — genuinely generic)

[LEARN:workflow] Spec-then-plan for complex/ambiguous tasks (>1h or >3 files): AskUserQuestion →
`quality_reports/specs/` with MUST/SHOULD/MAY → approval → plan. Catches ambiguity early.

[LEARN:workflow] Plans, specs, and session logs live on disk (`quality_reports/`), not just in
conversation, so they survive compression. Quality reports only at merge time.

[LEARN:workflow] Before compression: update MEMORY.md [LEARN] entries, ensure session log current,
save active plan to disk, document open questions.

## Collaboration preferences (Fred)

[LEARN:collab] Wants structured, precise, rigorous work even if slower. Visuals must be
publication-ready. Plan-first for non-trivial tasks; after plan approval, operate in contractor
mode (autonomous, return only on ambiguity/decisions). Check in more often during the first few
sessions while learning the workflow.

---

## [LEARN] Log

<!-- Append new corrections below. Most recent at bottom. Format: [LEARN:category] wrong → right -->
