# Pairs trading on KO / PEP

Take-home project for BlackSwanQuants: a mean-reversion strategy on the spread between Coca-Cola (KO)
and PepsiCo (PEP), and an honest check of whether it is a real signal or a lucky accident.

**Result in one line:** the strategy earns a Sharpe of 0.45 in-sample (1990-2014) but -0.50
out-of-sample (2015-2026). The other phases explain why: the two prices are not cointegrated, the hedge
ratio is unstable, and most of the in-sample profit comes from fitting the hedge ratio on the same data
and from one favourable period (1999-2004).

## Contents

| Path | What it is |
|---|---|
| `notebook.ipynb` | The full analysis, phase by phase (Phases 1-5), with comments under every result |
| `report.pdf` | Short writeup and AI usage disclosure |
| `data/raw/prices_raw.pkl`, `.csv` | Daily prices from Yahoo Finance, as downloaded (checkpoint) |
| `data/processed/prices_validated.pkl`, `.csv` | Cleaned and aligned KO and PEP prices |
| `data/raw/pep_short_interest.json`, `.csv` | PEP short interest from Nasdaq (last 12 months, checkpoint) |
| `data/raw/finra_short_interest.csv` | KO and PEP short interest from FINRA, 2017-2026 (manual export) |
| `results/trials.sqlite` | Registry of every backtest configuration run, with the time of its first run |

## How to run

Requirements: Python 3.12 and [uv](https://docs.astral.sh/uv/).

```
uv sync
uv run jupyter lab notebook.ipynb
```

Then run all cells from the top (Restart and Run All). No API key is needed.

- **Prices and Nasdaq short interest** are loaded from the checkpoints in `data/`, so the notebook runs
  offline and gives the same numbers every time. Deleting them triggers a new download, which may return
  different data (Yahoo revises history, Nasdaq keeps only a rolling 12 months).
- **FINRA short interest** cannot be downloaded by the code: it was exported manually from FINRA's
  Equity Short Interest page (symbols KO and PEP, all columns, Quick Export).
- **Simulations** use a fixed random seed (42), so results are reproducible.

## How the validation was kept honest

- The in-sample / out-of-sample split, the baseline rule and the parameter grid were fixed before
  downloading the data.
- Phase 1 exploration uses in-sample data only. The out-of-sample period was used once, for the
  baseline rule only.
- Every backtest is logged in `results/trials.sqlite`: 12 in-sample configurations (9 planned, 3 added
  after seeing the grid and labelled as such) and the baseline out-of-sample at three cost levels.
  The `first_run_at` column shows that all in-sample runs came before the out-of-sample run.
- Re-running the notebook updates the metrics and `last_run_at` of existing rows but keeps
  `first_run_at`, so the registry still records the original order.

## Phases and methods

| Phase | Method |
|---|---|
| 1. Data | Prices, KO/PEP ratio and rolling correlation, in-sample only |
| 2. Signal and validation | OLS hedge ratio, 60-day z-score (entry 2σ, exit 0, 5 bp costs), sensitivity grid, single out-of-sample test |
| 3. Stability | Medium: rolling hedge ratio and rolling Engle-Granger test |
| 4. Short interest | Nasdaq (PEP only) cross-checked with FINRA (KO and PEP) |
| 5. Simulation | Joint block bootstrap of in-sample returns (1,000 paths, blocks of 5, 10 and 20 days) |
