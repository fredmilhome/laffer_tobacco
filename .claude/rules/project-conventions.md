---
paths:
  - "scripts/R/**/*.R"
  - "data/**/*"
  - "results/**/*"
---

# Project Conventions: Tobacco Tax Laffer Simulation

Substantive invariants for the cigarette tax-revenue / consumption simulation. These bind code
review and any numeric claim. Full derivation and sources: [`project_context.md`](../../project_context.md).

## Calibration (the anchor)

- **Baseline target: `R_0 = R$17.75 bn/yr` (2019)** = IPI + PIS/COFINS + ICMS, national total (RFB via Divino 2021).
- Sub-national cigarette revenue is **not published** → reconstruct state revenue from the tax
  formula × state price × state quantity, calibrated so `Σ_s R_state[s] = R_0`.
- **The only calibration knob is the illicit (PC1) share.** Expand it proportionally (~+30pp over
  the VIGITEL raw rate) until model-implied revenue equals `R_0`. Do not tune elasticities or rates
  to hit the target.
- **Re-run calibration whenever the minimum price floor changes.** D12922 raises the floor to
  R$7.50, mechanically reclassifying R$5–7.50 buyers from PC2 into PC1 (illicit). Stale calibration
  contaminates the categories.

## Tax stack (current, post-D12922/2026)

```
T_total[k,s] = tau_s + t_av_IPI*alpha*p + t_PIS_COFINS*p + t_ICMS[s]*p
p            = (p_0 + tau_s) / (1 - t_av_IPI*alpha - t_PIS_COFINS - t_ICMS[s])
p_floor      = max(minimum_price, price_implied_by_IPI_formula)
theta[k,s]   = T_total[k,s] / p[k,s]          # tax burden share, ~0.69–0.83
```

Key values (verify against `00_config.R`, not memory): `tau_s` = R$2.25/vintena (→ R$3.50 from
1 Aug 2026); `t_av_IPI` = 66.70%, `alpha` = 0.15 (⇒ 10% effective ad valorem on retail);
`t_PIS_COFINS` ≈ 10.98%; `t_ICMS[s]` ≈ 25–35% by state (Ribeiro & Pinto 2019 — verify for updates);
minimum price R$6.50 → **R$7.50 from 1 May 2026**.

## Demand response

- **Exact CES is the default for Phase 1 onward:** `pct_delta_Q = (p_new/p_old)^ε − 1`.
- The **linearized** form `pct_delta_Q = ε · pct_delta_p` is for **replicating Divino et al. 2021 only.**
  For price changes >20% it materially overstates the consumption fall (~12pp in the paper). Set via
  `DEMAND_FUNCTIONAL_FORM` in `00_config.R`; never silently switch forms.
- Apply elasticities **cell-by-cell** over region × price category. **No within-legal switching**
  (PC2↔PC3↔PC4 prohibited) and **illicit share held fixed** through reform scenarios.

## Elasticities (provenance matters)

- Region × PC total elasticities: Divino et al. 2021 Table (hardcoded for Phase 1).
- Intensive (licit, state-avg prices) `ε_int ≈ −0.87`; participation (whole market) `ε_part ≈ −0.48`
  (Divino 2024, *Tobacco Control*). Total licit `ε_total ≈ −1.35` **requires the attribution
  assumption** that whole-market participation applies to the licit market — valid only if `ε^{I,L} ≈ 0`.
- **Laffer peak exists only if `|ε_total| > 1`.** With intensity alone (`ε ≈ −0.87`, inelastic)
  there is **no finite peak** under constant elasticity. `theta* = 1/|ε|`; at `ε ≈ −1.35`,
  `theta* ≈ 0.74` vs `theta_0 ≈ 0.72` → Brazil ≈ at the peak. Report this sensitivity explicitly.

## Cross-elasticity stress test (Phase 3 contract)

- Paper estimate `ε^{I,L} ≈ 0` (Divino 2025 JAE) — licit price rises do **not** grow the illicit
  market. Asymmetric: `ε^{L,I} > 0`. **This is the result being stress-tested**, not an assumption to inherit.
- Re-run under `CROSS_ELASTICITY_VALUES = [0.0, 0.3, 0.6]`. When `ε^{I,L} > 0`:
  `ε_eff = ε_own + ε^{I,L} · (Q_illicit/Q_licit)`, which raises `|ε|`, lowers `theta*`, and can push
  Brazil past the peak. Also scale the participation-margin attribution down accordingly.

## Config-not-hardcoded (enforced)

These live in `scripts/R/00_config.R` only: `BASELINE_YEAR`, `IS_RATE_GRID` (0–40% in 5pp steps),
`CROSS_ELASTICITY_VALUES`, `DEMAND_FUNCTIONAL_FORM` (`"exact_CES"` | `"linearized"`),
`REVENUE_SCOPE` (`"total"` | `"federal"`), `MARKUP_SCENARIOS` (`min_profit`, `avg_markup`, `max_markup`).
IS stacking vs. PIS/COFINS replacement is a config switch — parse the rule from `regulations/PLP68`.

## Cross-references

- [`single-source-of-truth.md`](single-source-of-truth.md) — the derivation chain.
- [`project_context.md`](../../project_context.md) — full literature, data, and pipeline spec.
