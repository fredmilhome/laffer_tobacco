# Project Context: Estimating Cigarette Tax Revenue and Consumption Responses in Brazil

*Summary of research design decisions, literature, data landscape, and pipeline specification. Written for Claude Code navigating this repository.*

---

## 1. Research Goals

Two simulation targets:

**(i) Recent regulatory changes (Decreto 12.922/2026)**
- Specific IPI tax: R$1.50 (pre-2024) → R$2.25 (Nov 2024) → **R$3.50 from 1 Aug 2026**
- Minimum retail price per vintena: R$5.00 → R$6.50 (Sep 2024) → **R$7.50 from 1 May 2026** (already in effect)
- Simulate the consumption and revenue response to each step change
- Note: minimum price and specific tax interact — for each price category the binding floor is `max(minimum_price, IPI_formula_implied_price)`

**(ii) Imposto Seletivo (IS) reform scenarios**
- EC 132/2023 created IS as part of the broader tax reform; PLP 68/2024 implements it
- IS rates on tobacco not yet set — sweep over a grid of ad valorem rates (suggested: 0% to 40% in 5pp steps)
- IS may stack on top of current IPI during transition, or partially replace PIS/COFINS
- Three industry response scenarios per rate (following Divino et al. 2021): minimum markup, average markup, maximum markup

Both goals require: (a) a calibrated baseline, (b) price category × region elasticities, (c) the tax formula, (d) industry markup-level assumptions.

---

## 2. Core Literature

### Divino, Ehrl, Candido & Valadão (2021) — IJERPH 18(19): 10376
**The methodology paper.** Simulates PL 3887-2020 (CBS replacing PIS/COFINS) under three industry scenarios.

**Data**: VIGITEL 2018+2019 pooled (5,314 smokers; baseline behavior); PNAD 2008 + PNS 2013 (elasticity estimation); IBGE population by state; Receita Federal national revenue total; ICMS rates from Ribeiro & Pinto (2019).

**Price categories**: PC1 = below official minimum price floor (illicit); PC2/PC3/PC4 = tertiles of legal market by price.

**Calibration**: illicit share (PC1) is expanded proportionally (~30pp added) until model-implied revenue equals the observed R$17.75bn for 2019. Receita Federal does not publish sub-national cigarette tax revenue, so state-level revenue is reconstructed from the tax formula applied to state prices × state quantities.

**Elasticity estimation**: two-part model. Step 1: probit on smoking indicator (prevalence elasticity). Step 2: log-log OLS on consumption conditional on smoking (intensity elasticity). State average prices replace individual prices to avoid endogeneity. Elasticities estimated by 5 regions × 4 price categories.

**Elasticity table (total = prevalence + intensity)**:

| Region | PC1 | PC2 | PC3 | PC4 |
|---|---|---|---|---|
| Northeast | −0.86 | −0.68 | −0.62 | −0.57 |
| North | −0.73 | −0.68 | −0.50 | −0.48 |
| Southeast | −0.56 | −0.68 | −0.46 | −0.42 |
| South | −0.51 | −0.66 | −0.40 | −0.39 |
| Midwest | −0.69 | −0.67 | −0.42 | −0.47 |

Prevalence elasticity: −0.21 to −0.26 across regions (not category-specific in this paper).

**Consumption change formula**: linearized CES, applied cell-by-cell:
```
pct_delta_Q[k,r] = epsilon[k,r] * pct_delta_p[k,r]
```
**Warning**: this is a first-order approximation. For large price changes (>20%) the exact CES formula `(p_new/p_old)^epsilon - 1` gives meaningfully different results. The paper uses the linearization throughout.

**Markup scenarios**: key variable is post-tax margin per pack: `M[k,s] = p[k,s] - T[k,s]`. Production costs assumed constant and uniform across states.
- **Scenario I (minimum price)**: find lowest `p_k` (national, uniform) where `M >= 0` for all states. Binding state is highest-ICMS state. For CBS reform gives R$8.40 for PC2 and PC3 (they converge).
- **Scenario II (average markup)**: set `p_k` such that post-reform margin equals smoker-share-weighted average of baseline margins across states.
- **Scenario III (maximum markup)**: set `p_k` to preserve the highest baseline margin observed in any state.

Prices under the reform become uniform nationally (because CBS/IS tax base = highest national price per brand → no incentive to price-differentiate by state).

**No switching between price categories allowed.** Consumers within each PC either reduce intensity or quit; they don't move to cheaper categories. Illicit market share held fixed throughout reform scenarios.

**Full scenario results (Table 2)**:

| | Baseline | Scenario I | Scenario II | Scenario III |
|---|---|---|---|---|
| Revenue (R$ bn/yr) | 17.75 | 23.20 (+30.7%) | 21.92 (+23.5%) | 20.53 (+15.7%) |
| PC2 price | R$5.38 | R$8.40 | R$10.03 | R$11.06 |
| PC2 consumption Δ | — | −38.1% | −58.6% | −71.5% |
| PC3 price | R$7.90 | R$8.40 | R$13.15 | R$14.87 |
| PC3 consumption Δ | — | −3.2% | −33.5% | −44.6% |
| PC4 price | R$12.84 | R$15.23 | R$19.42 | R$23.38 |
| PC4 consumption Δ | — | −9.6% | −25.5% | −40.5% |

---

### Divino, Ehrl, Candido & Valadão (2024) — *Tobacco Control* 33(Suppl 2): s122–s127
**The elasticity paper.** Estimates licit and illicit own-price elasticities by income quartile and age cohort.

**Data**: PNS 2013 + PNS 2019 pooled (n ≈ 10,300 smokers for conditional models; n ≈ 122,900 for participation probit).

**Illicit identification**: PNS 2013 — purchases below R$3.50 floor = illicit. PNS 2019 — ANVISA brand registry (more accurate; some illicit brands sold above floor).

**Endogeneity fix**: state-average prices replace individual prices. Identifies from cross-state ICMS variation and logistics differentials.

**Key estimates (state-average prices, preferred specification)**:
- Licit conditional (intensive margin): −0.85 to −0.99 by age cohort; −0.87 to −0.90 by income quartile
- Illicit conditional: −0.20 to −0.38 (consistently lower than licit)
- Participation (whole market, from probit): −0.44 to −0.51

**Critical gap for Laffer estimation**: participation probit does not distinguish licit vs. illicit. Total unconditional licit elasticity must be approximated as `epsilon_int_licit + epsilon_part_total`. The zero cross-price result (see below) justifies this: if licit price rises cause no switching to illicit, all extensive-margin response is true exit, so whole-market participation elasticity applies to the licit market.

Summing: `epsilon_total_licit ≈ −0.87 + (−0.48) ≈ −1.35`. This is elastic (|ε| > 1), implying a finite Laffer peak at `theta* = 1/1.35 ≈ 0.74`, close to Brazil's current burden share `theta_0 ≈ 0.72`.

---

### Divino, Ehrl, Candido, Valadão & Rodriguez-Iglesias (2025) — *Journal of Applied Economics*
**The cross-price elasticity paper.**

Key finding: `epsilon^{I,L} ≈ 0` (licit price increases do not increase illicit demand — statistically indistinguishable from zero). But `epsilon^{L,I} > 0` (illicit price rises push some smokers to licit brands).

This is **asymmetric**: raising licit prices does not grow the illicit market, but cracking down on illicit prices does benefit the licit market. This result was assumed (not estimated) in Divino et al. 2021; this paper provides ex-post validation.

**Stress-testing this result is a project goal.** The zero result is somewhat surprising and has major policy implications (it removes the tobacco industry's main Laffer argument). Test under three assumptions:
- `epsilon^{I,L} = 0` (paper estimate)
- `epsilon^{I,L} = 0.3` (moderate switching)
- `epsilon^{I,L} = 0.6` (high switching)

When `epsilon^{I,L} > 0`, effective licit own-price elasticity is:
```
epsilon_eff = epsilon_own + epsilon^{I,L} * (Q_illicit / Q_licit)
```
This increases |ε|, lowers θ*, and potentially pushes Brazil past the Laffer peak.

---

### Divino et al. (2024) — Tobacconomics Working Paper 24/2/3
**The IS simulation paper.** Simulates PEC 45-A / IS reform at different selective tax rates for tobacco.

Calibrates to 2019 baseline. Finds IS rate of 171% needed to match baseline revenue; 232% to ensure no state loses revenue. Uses PNS 2019 + VIGITEL for behavior.

---

## 3. Tax Formula

### Current structure (post D12922/2026)

**IPI** (special regime, used by all major manufacturers):
```
T_IPI = tau_s + t_av_IPI * alpha * p
```
where:
- `tau_s` = alíquota específica = R$2.25/vintena (Nov 2024 – Jul 2026), **R$3.50 from 1 Aug 2026**
- `t_av_IPI` = 66.70% (ad valorem rate)
- `alpha` = 0.15 (IPI ad valorem applies to 15% of retail price)
- Effective IPI ad valorem on retail = 66.70% × 15% = 10%

**PIS/COFINS**: approximately 10.98% of retail price (substitution tributária — paid by manufacturer)

**ICMS**: varies by state (`t_ICMS^s`); ranges roughly 25%–35%; from Ribeiro & Pinto (2019) — verify for updates.

**Consumer price from tax stack**:
```
p = (p_0 + tau_s) / (1 - t_av_IPI*alpha - t_PIS_COFINS - t_ICMS^s)
```

**Minimum retail price**: R$6.50/vintena (Sep 2024 – Apr 2026), **R$7.50 from 1 May 2026**. The effective floor is:
```
p_floor = max(minimum_price, price_implied_by_IPI_formula)
```

**Total tax per pack**:
```
T_total[k,s] = tau_s + t_av_IPI*alpha*p + t_PIS_COFINS*p + t_ICMS^s * p
```

**Tax burden share**:
```
theta[k,s] = T_total[k,s] / p[k,s]
```
Ranges from ~0.69 (high-price brands) to ~0.83 (low-price brands in high-ICMS states).

### Revenue decomposition
- **R$17.75 bn (2019)** = IPI + PIS/COFINS + ICMS (national total, from Receita Federal)
- IPI and PIS/COFINS: paid by manufacturer at factory location (Santa Cruz do Sul, RS for Souza Cruz/BAT ~78%; Philip Morris ~12%). **No public geographic breakdown available.**
- ICMS: state revenue; CONFAZ publishes total state ICMS but **not broken down by product**.
- State-level revenue must be **reconstructed** from tax formula × price × quantity by state (as Divino et al. do).

---

## 4. Laffer Curve Framework

The normalized CES revenue function anchored at baseline `(theta_0, R_0)`:

```
R(theta) / R_0 = (theta / theta_0) * ((1 - theta_0) / (1 - theta/rho))^(1 + epsilon_hat)
```

Revenue-maximizing burden share: `theta* = 1 / |epsilon_hat|`

With `epsilon_hat ≈ −1.35`: `theta* ≈ 0.74`, very close to `theta_0 ≈ 0.72`. Brazil is approximately at the Laffer peak under these estimates — but the result is highly sensitive to whether the participation margin is included.

**Key sensitivity**: if only intensity elasticity is used (`epsilon ≈ −0.87`, inelastic), under constant-elasticity demand there is **no finite Laffer peak** — revenue is monotonically increasing. The existence of a peak depends entirely on whether `|epsilon_total| > 1`, which requires including the participation margin.

---

## 5. Data Sources

### Available public data

| Dataset | Granularity | Format | URL pattern |
|---|---|---|---|
| VIGITEL (annual, 2006–2024) | Individual, 26 capitals + DF | Stata .dta | `svs.aids.gov.br/download/Vigitel/` |
| PNS 2013 | Individual, national | Stata/CSV | `ftp.ibge.gov.br/PNS/2013/` |
| PNS 2019 | Individual, national | Stata/CSV | `ftp.ibge.gov.br/PNS/2019/` |
| PNAD 2008 | Individual, national | Stata/CSV | `ftp.ibge.gov.br/.../PNAD.../2008/` |
| IBGE population by state | State, annual | CSV | IBGE Censo 2022 / PNAD Contínua |
| IPCA tobacco sub-index | Monthly, regional | CSV | SIDRA table 1419, item 35007 |
| RFB IPI + PIS/COFINS (cigarettes) | Monthly, national | CSV/PDF | `gov.br/receitafederal/.../arrecadacao-de-tributos-federais-[year]` |
| RFB production volumes | Annual, national | PDF table | `gov.br/receitafederal/.../producao-de-cigarros-no-brasil-[year]` |
| CONFAZ ICMS boletim | State, monthly | CSV | `confaz.fazenda.gov.br/boletim-de-arrecadacao-dos-tributos-estaduais` |

### Does not exist publicly
- Sub-national IPI or PIS/COFINS broken down by product category
- State-level ICMS from cigarettes specifically (must be reconstructed)
- State-level production distribution (nearly all production in RS anyway)

### State ICMS rates
Source: Ribeiro & Pinto (2019), Red Sur/Tobacconomics. Must be verified for changes since 2019; a few states have updated rates. Build as `data/processed/state_icms_rates.csv` with columns: `state, UF_code, icms_rate, source, year_valid`.

---

## 6. Pipeline Specification

### Phases

**Phase 1 — Direct estimates** (hardcoded elasticities from Divino et al. 2024 Table 2–4)
- Calibrate 2019 baseline (matching R$17.75bn via illicit share adjustment)
- Simulate D12922 step changes (R$2.25→R$3.50 specific; R$6.50→R$7.50 floor)
- Simulate IS rate grid under three markup scenarios
- Output: revenue and consumption tables by scenario

**Phase 2 — Own elasticity estimates** (from PNS 2013 + 2019)
- Two-part model: probit (participation) + log-log OLS (intensity)
- State-average prices as instrument
- Estimate by region × price category
- Feed into same simulation pipeline

**Phase 3 — Cross-elasticity stress test**
- Re-run Phase 1 under three values of `epsilon^{I,L}`: 0, 0.3, 0.6
- Show how Laffer peak position and revenue estimates shift

### Core simulation loop (`scripts/R/05_simulate.R`)

```r
for (scenario in c("min_profit", "avg_markup", "max_markup")) {
  for (rate in cfg$IS_RATE_GRID) {
    p_new      <- compute_new_price(tax_params, markup_rule = scenario)  # by [k, s]
    pct_dp     <- (p_new - p_base) / p_base                              # by [k, r]
    pct_dQ <-
      if (cfg$DEMAND_FUNCTIONAL_FORM == "exact_CES") (p_new / p_base)^epsilon - 1  # default Phase 1+
      else epsilon * pct_dp                                                        # linearized: replication only
    Q_new <- Q_base * (1 + pct_dQ)
    T_new <- compute_tax(p_new, tau_s_new, t_av, t_PIS, t_ICMS)          # vectorized over s
    R_new <- T_new * Q_new * N_smokers                                   # by [k, s]
  }
}
```

Aggregate over `k` (price categories) and `s` (states) to get total revenue and consumption.
Implement cells as a long `data.frame`/`tibble` keyed on `(k, s)` and use grouped `dplyr`
summaries rather than nested loops where practical; the loop above is the conceptual contract.

### Configuration (`scripts/R/00_config.R`)

Parameters to expose as config (not hardcoded). Source this at the top of every stage as `cfg`:
```r
cfg <- list(
  BASELINE_YEAR         = 2019,                          # or 2023
  IS_RATE_GRID          = seq(0, 0.40, by = 0.05),       # 0% to 40% in 5pp steps
  CROSS_ELASTICITY_VALUES = c(0.0, 0.3, 0.6),
  DEMAND_FUNCTIONAL_FORM = "exact_CES",                  # or "linearized" (Divino 2021 replication)
  REVENUE_SCOPE         = "total",                        # or "federal"
  MARKUP_SCENARIOS      = c("min_profit", "avg_markup", "max_markup")
)
```

---

## 7. Known Issues and Modeling Choices

**Linearization error**: for price changes above ~20%, the linearized CES formula overstates consumption falls relative to exact CES. Scenario I for PC2 involves a 56% price increase — the error is material (~12pp in the paper's results). Default to exact CES for Phase 1 onwards; keep linearized version for replication of Divino et al.

**Participation margin attribution**: the probit in Divino et al. 2024 does not separate licit vs. illicit participation. The justification for using whole-market `epsilon_part` for the licit market is `epsilon^{I,L} ≈ 0` — this is the result being stress-tested. Under `epsilon^{I,L} > 0`, the participation margin should be scaled down proportionally.

**Price category boundaries shift**: D12922 raises minimum price to R$7.50. Under Divino et al.'s definition, PC1 = below floor. This mechanically reclassifies some current PC2 buyers (those paying R$5–7.50) as PC1 (illicit). The calibration must be re-run with updated floor to avoid category contamination.

**No within-legal cross-elasticities**: the model prohibits PC2↔PC3↔PC4 switching. This is conservative — if consumers can downgrade brands (e.g., PC3→PC2) when PC3 prices rise sharply, the aggregate consumption fall is smaller and the revenue effect is different. This is an unresolved gap in the literature.

**Revenue scope and IS stacking**: if IS stacks on top of IPI (transition period), compute `T_total = T_IPI + T_PIS_COFINS + T_ICMS + T_IS`. If IS replaces PIS/COFINS, adjust accordingly. The PLP 68/2024 specifies the transition rules — parse this from the HTML regulatory file.

**State-level revenue reconstruction**: since sub-national revenue data doesn't exist, state revenue is computed as:
```
R_state[s] = sum_k T[k,s] * Q[k,s] * N[k,s]
```
The calibration target is national: `sum_s R_state[s] = R_0`. Calibrate illicit share so this holds.

---

## 8. Files in This Repo

```
regulations/         HTML files of key decrees (D12922, D12127, D7555, PLP68, EC132)
papers/              .md transcriptions of key papers (see §2 above)
data/raw/            VIGITEL, PNS, PNAD, IBGE, RFB data (not committed — see data/README.md)
data/processed/      state_icms_rates.csv, baseline_calibrated.csv
scripts/R/           Pipeline scripts (00_config.R through 07_results.R; outputs in scripts/R/_outputs/)
results/             Output tables/figures by scenario
```

---

## 9. Quick Reference: Key Numbers

| Parameter | Value | Source | Notes |
|---|---|---|---|
| `R_0` (2019) | R$17.75 bn/yr | RFB via Divino 2021 | IPI + PIS/COFINS + ICMS |
| Specific tax (current) | R$2.25/vintena | D12127/2024 | Until Jul 2026 |
| Specific tax (Aug 2026) | R$3.50/vintena | D12922/2026 | New |
| Minimum price (current) | R$7.50/vintena | D12922/2026 | In effect since May 2026 |
| Ad valorem IPI on retail | 10% (= 66.7% × 15%) | D7555/2011 | Unchanged |
| PIS/COFINS on retail | ~10.98% | Lei 9.715/1998 | Via substituição tributária |
| ICMS range | 25%–35% | Ribeiro & Pinto 2019 | Verify for updates |
| `theta_0` (2019) | ~0.69–0.73 | Divino 2024 WP | By price category |
| `epsilon_int_licit` | −0.87 to −0.90 | Divino 2024 TC | State-avg prices, by income quartile |
| `epsilon_part` | −0.44 to −0.51 | Divino 2024 TC | Whole market |
| `epsilon_total_licit` | ≈ −1.35 | Approximation | Requires attribution assumption |
| `epsilon^{I,L}` | ≈ 0 | Divino 2025 JAE | Key result to stress-test |
| Illicit share (2019) | ~30pp above VIGITEL raw | Divino 2021 calibration | Calibrated to match R_0 |
| Illicit share range | 19% (AM) – 53% (MS) | Divino 2021 Fig 1 | State-level |
| Production record | 4.5 bn packs (2023) | RFB 2024 | +10% YoY |
