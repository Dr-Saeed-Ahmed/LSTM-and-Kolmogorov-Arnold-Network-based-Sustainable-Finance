# LSTM and Kolmogorov-Arnold Network based Sustainable Finance

This package reproduces every number, table, and figure in the revised manuscript
from the raw returns file. It was built to replace the earlier mixed 6-asset/13-asset
pipeline with one consistent, verified workflow.

## Folder structure
```
data/Returns_Data.csv        # raw daily log-returns (13 assets, Jun 2018–May 2026)
code/dy_core.py              # shared Diebold–Yilmaz functions (VAR, GFEVD, TCI, BIC)
code/01_data_cleaning.py     # bad-tick detection + cleaning (audit trail saved)
code/02_static_rolling_dy.py # static + rolling-window DY connectedness (13 assets)
code/03_tvpvar.py            # TVP-VAR (Kalman filter, forgetting factors) + sensitivity
code/04_qvar_garch.py        # quantile VAR tail connectedness + GARCH(1,1) volatilities
code/05_ml_forecasting.py    # forecasting feasibility: persistence/AR/Ridge/LSTM/LSTM-KAN + DM tests
code/06_make_figures.py      # regenerates all figures from results CSVs
results/                     # all output tables/series (CSVs)
figures/                     # all publication figures (300 dpi PNG)

```

## How to run
```bash
pip install -r requirements.txt
cd code
python 01_data_cleaning.py
python 02_static_rolling_dy.py    
python 03_tvpvar.py              
python 04_qvar_garch.py           
python 05_ml_forecasting.py       
python 06_make_figures.py
```

## Data notes
- The raw file uses the column name `NCLR` for the nuclear-energy ETF; the scripts
  rename it to the canonical ticker `NLR` (01_data_cleaning.py). All results,
  figures, and the manuscript use `NLR`.
- Eight physically implausible ETF returns (|r| > 20%, including one below −100%)
  are neutralized by 01_data_cleaning.py; see results/cleaning_audit.csv and
  Section 3.1 of the paper. Verify these prints against your data vendor before
  reusing the raw file elsewhere.

## Key settings (as reported in the paper)
- VAR lag p = 2 (BIC-selected: p=2 → BIC 78,954 < p=1 → 79,789)
- FEVD horizon H = 10; rolling window W = 200 trading days
- TVP-VAR: forgetting factors λ = κ = 0.99; reported for the 6 primary assets
  (the 13-asset forgetting-factor filter is under-determined — see Section 4.3
  and results/tvpvar_lambda_sensitivity.csv)
- QVAR: quantiles τ ∈ {0.05, 0.25, 0.50, 0.75, 0.95}
- ML: 60/20/20 chronological split; features standardized on training data only;
  5 seeds per deep model; Diebold–Mariano tests vs the persistence benchmark

