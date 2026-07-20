# Deep Reinforcement Learning for Cryptocurrency Perpetual Futures Trading

Companion code and data for the MSc dissertation *Deep Reinforcement Learning for
Cryptocurrency Perpetual Futures Trading* (NOVA Information Management School,
Universidade NOVA de Lisboa, 2026) by Nuno Vieira.

The repository contains the complete, executable pipeline behind the dissertation:
data acquisition, data quality auditing, the full experiment, and the analysis that
reproduces every results table in the thesis.

## Contents

```
1_DataDownload.ipynb                          Downloads the raw Binance archives and builds the aligned panel
2_DataQualityAudit.ipynb                      Data quality audit backing the data chapter
3_Experiment.ipynb                            The experiment: environment, agents, 1,680-run ablation sweep
4_Analysis.ipynb                              Reproduces every results table of the dissertation
data/                                         Raw archive cache, merged per-feed CSVs, processed panel
artifacts_thesis_experiment_trainvaltest/     Sweep outputs (run-level results, manifest)
sweep.log                                     Log of the original full sweep
```

## The experiment in brief

Three deep reinforcement learning algorithms (PPO, A2C, RecurrentPPO) are trained to
trade Binance USDT-margined perpetual futures on BTC, ETH, SOL, and XRP, at hourly
resolution, in a custom Gymnasium environment with funding settlement,
maintenance-margin liquidation, and transaction costs. A 2⁴ factorial ablation varies
four design axes — encoder, leverage, reward, episode scheme — across five targets
(one portfolio agent and four asset-specific agents). A two-phase protocol first runs
480 selection trainings (seed 42, winner by validation Sharpe), then 1,200 final
trainings (16 cells × 3 algorithms × 5 targets × seeds 5/10/15/20/25). Data spans
1 January 2022 – 30 April 2026; the held-out test window is September 2025 – April 2026.

## The notebooks

**1_DataDownload** — downloads the 1,040 monthly ZIP archives (futures, spot, and
mark-price klines; premium index; funding rates) from the Binance public data
repository (`data.binance.vision`), merges them into 16 per-symbol CSVs, and builds
the aligned hourly panel (`data/processed/aligned_panel_20220101_20260430.csv`,
151,536 rows). Switches at the top control download, export, and panel rebuild.

**2_DataQualityAudit** — display-only audit of the raw archives and the panel:
embedded header rows in the raw files, mixed millisecond/microsecond timestamp
encodings, missing hours, coverage per feed, and the November 2022 SOL funding
anomaly (FTX collapse: emergency 2-hour funding intervals with rates pinned at the
−2% cap). Every data-cleaning claim in the dissertation is evidenced here.

**3_Experiment** — self-contained: the trading environment, the attention–LSTM and
flat feature extractors, the ablation grid, the selection and final phases, and the
parallel sweep runner. The full sweep is guarded by a `RUN_FULL_SWEEP` switch;
embedded smoke tests validate all mechanics in minutes. The original sweep ran in
about 2.3 hours on an AWS c7i.24xlarge (96 vCPUs, 90 workers); `sweep.log` is its log.

**4_Analysis** — reads only the sweep artifacts and the panel, and reproduces every
results table of the dissertation — Chapter 5 (Tables 5.1–5.16, including the
buy-and-hold baselines and the Wilcoxon signed-rank tests) and Appendix A
(Tables A.1–A.3) — inline, in the thesis's own format. Writes no files; runs in seconds.

## Artifacts

`artifacts_thesis_experiment_trainvaltest/tables/`:

- `summary_ablation.csv` — 1,200 rows, one per final run: configuration plus the full
  train/test metric suite. Source of every number in the results chapter.
- `tuning_log.csv` — the 480 selection-phase runs.
- `per_cell_summary.csv`, `axis_effect_summary.csv` — aggregates.
- `manifests/manifest.json` — sweep configuration snapshot.

## Requirements

Python ≥ 3.10 with `stable-baselines3` (2.x), `sb3-contrib`, `gymnasium`, `torch`,
`pandas`, `numpy`, `scipy`, `matplotlib`, `joblib`, and `jupyter`.

## Reproducibility

- Path constants at the top of each notebook are anchored to the project folder;
  point them at your local copy before running.
- `4_Analysis` verifies every published table from the shipped artifacts — no
  retraining required.
- All seeds are fixed (selection seed 42; final seeds 5, 10, 15, 20, 25) and passed to
  Stable-Baselines3, which seeds Python, NumPy, and PyTorch.
- The data pipeline is verified end-to-end: re-downloading the Binance archives with
  `1_DataDownload` reproduces the shipped panel value-for-value.

## Data source

All market data originate from the Binance public data repository
(https://data.binance.vision). The `data/` folder redistributes them in merged and
aligned form for convenience; `1_DataDownload` rebuilds everything from the original
archives.

## Citation

If you use this code or data, please cite the dissertation:

> Vieira, N. (2026). *Deep Reinforcement Learning for Cryptocurrency Perpetual
> Futures Trading*. MSc dissertation, NOVA Information Management School,
> Universidade NOVA de Lisboa.

A DOI for this repository is provided via Zenodo: (add DOI badge after the first
GitHub release is archived).
