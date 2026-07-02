# Pairs Trading with Python

A statistical-arbitrage research notebook that scans a basket of stocks for cointegrated
pairs, builds a mean-reversion signal from the price ratio of a chosen pair, and backtests
a simple long/short strategy on it.

## Contents

- `pairstrading.ipynb` — the full pipeline, from data download to backtest results.
- `requirements.txt` — Python dependencies.

## Setup

```
pip install -r requirements.txt
```

Then open `pairstrading.ipynb` in Jupyter / VS Code and run all cells top to bottom.
Data is pulled live from Yahoo Finance via `yfinance`, so an internet connection is required.

## What the notebook does

1. **Download data** — daily close prices (2020–2025) for a basket of major US bank tickers
   (JPM, BAC, WFC, C, GS, MS, USB, PNC, TFC, COF), chosen because same-sector names sharing
   macro drivers (rate cycle, credit conditions) are more likely to be genuinely cointegrated.
2. **Train/test split** — splits the price history 70/30 chronologically, before any model
   fitting, so pair selection and hedge-ratio estimation can't see the test period.
3. **Cointegration scan** — runs the Engle-Granger cointegration test on every pair in the
   basket (using train data only) and heatmaps the resulting p-values.
4. **Pair selection** — among pairs significant on the full training set, splits training data
   again (80/20) into a fit/validation slice and picks the pair that stays cointegrated on
   both, without ever touching the test period. Fits a hedge ratio via OLS on the chosen pair,
   and checks the spread for stationarity (ADF test) and out-of-sample cointegration.
5. **Signal construction** — builds a rolling z-score of the price ratio (5-day vs. 60-day
   moving average) as the mean-reversion signal.
6. **Backtest** — a `trade()` function that opens a self-financed long/short position when the
   z-score crosses ±1, closes it when the z-score reverts inside ±0.75, marks the position to
   market daily, and reports total P&L, trade count, annualized Sharpe ratio, and max drawdown.

## Known limitations

- No transaction costs, slippage, or margin requirements are modeled.
- The hedge ratio is estimated once on train data and held fixed (no rolling re-estimation).
- Position sizing is a single hedge unit at a time, not scaled to a capital base.
- Pair selection tests 45 pairs at p < 0.05 with no multiple-comparisons correction, so some
  "significant" pairs may be false positives. In practice, pairs that look cointegrated
  in-sample (even robust across a fit/validation split) have not held up on the true test
  period in this basket — treat the cointegration scan as a screen, not a trading signal.

This is a research/learning notebook, not a production trading system.
