---
paths:
  - "scripts/R/**/*.R"
  - "data/processed/**/*"
  - "results/**/*"
---

# Single Source of Truth: Enforcement Protocol

**The `scripts/R/` pipeline and its generated outputs are the authoritative source for every
number, table, and figure in this project.** Everything downstream is derived. Never edit a
derived artifact by hand; always propagate from the source script and re-run.

## The derivation chain

```
data/raw/*               (immutable inputs — VIGITEL, PNS, PNAD, IBGE, RFB, CONFAZ)
  └─ 01_load.R      → load + harmonize
     └─ 02_clean.R  → data/processed/*.csv  (clean panel, state_icms_rates, baseline)
        └─ 03_analyze.R → scripts/R/_outputs/*.rds   (calibration, elasticities, simulation)
           ├─ 04_tables.R  → results/tables/*        (derived)
           └─ 05_figures.R → results/figures/*       (derived)
              └─ paper / slides numbers              (derived — must cite an .rds)

NEVER edit a results/ table, a figure, or a paper number independently.
ALWAYS change the upstream script (or 00_config.R), re-run, and let outputs regenerate.
```

`scripts/R/00_config.R` is the **parameter source of truth**: baseline year, IS-rate grid,
cross-elasticity values, demand functional form, revenue scope, markup scenarios, and the
calibration target all live there — not inline in 01–07. Changing a parameter means editing
`00_config.R` and re-running, never patching a downstream value.

## Freshness protocol (MANDATORY)

Before trusting any number in `results/`, the paper, or a slide:

1. Confirm the `.rds` it derives from is **newer** than every script upstream of it.
2. If any upstream script (or `00_config.R`) changed since the `.rds` was written, **re-run** the
   affected stages before using the number.
3. Re-run the full pipeline (`Rscript scripts/R/00_run_all.R`) before any commit that changes
   analysis logic, parameters, or input data.

## Reproducibility checklist

```
[ ] Every results/ table and figure traces to a named scripts/R/_outputs/*.rds
[ ] No parameter hardcoded outside 00_config.R
[ ] Calibration reproduces R_0 = R$17.75 bn (2019) — see project-conventions.md
[ ] Seeds set for any stochastic step; RNG kind recorded
[ ] No derived artifact edited by hand since its source last ran
[ ] Numbers cited in papers/ or slides match the current .rds (run /audit-reproducibility)
```

## Cross-references

- [`.claude/rules/project-conventions.md`](project-conventions.md) — the substantive invariants.
- [`.claude/rules/cross-artifact-review.md`](cross-artifact-review.md) — paper ↔ code dependency graph.
- [`.claude/rules/replication-protocol.md`](replication-protocol.md) — numeric tolerance contract.
