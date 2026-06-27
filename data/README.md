# Data

**Raw data is NOT committed.** VIGITEL / PNS / PNAD microdata are large and
redistribution-restricted; RFB/CONFAZ files are re-downloadable. Keep `data/raw/` local; commit
only the small derived files in `data/processed/` (e.g. `state_icms_rates.csv`,
`baseline_calibrated.csv`).

## Layout

```
data/
├── raw/         # NOT committed — download per the sources below
└── processed/   # small derived CSVs — committed
```

## Sources (see project_context.md §5 for the full table)

| Dataset | Granularity | Source |
|---|---|---|
| VIGITEL (2006–2024) | Individual, 26 capitals + DF | `svs.aids.gov.br/download/Vigitel/` |
| PNS 2013 / 2019 | Individual, national | `ftp.ibge.gov.br/PNS/` |
| PNAD 2008 | Individual, national | IBGE FTP |
| IBGE population by state | State, annual | IBGE Censo 2022 / PNAD Contínua |
| IPCA tobacco sub-index | Monthly, regional | SIDRA table 1419, item 35007 |
| RFB IPI + PIS/COFINS (cigarettes) | Monthly, national | `gov.br/receitafederal/.../arrecadacao-...` |
| RFB production volumes | Annual, national | `gov.br/receitafederal/.../producao-de-cigarros-...` |
| CONFAZ ICMS boletim | State, monthly | `confaz.fazenda.gov.br/boletim-...` |

## Does not exist publicly (must be reconstructed)

- Sub-national IPI / PIS/COFINS by product category
- State-level ICMS from cigarettes specifically (reconstruct from the tax formula)
- State-level production distribution (≈ all production in RS)

`data/processed/state_icms_rates.csv` columns: `state, UF_code, icms_rate, source, year_valid`.
Verify Ribeiro & Pinto (2019) rates for post-2019 changes.
