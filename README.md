# Polymarket — Next CEO of NVIDIA? Forecast

Offline 4-hour-ahead Yes-price quantile forecaster for the Polymarket
"Next CEO of NVIDIA?" event. `node main.ts` (Node 22, stdlib only, no
network, no clock, no randomness) reads every `*.in` in
`$MAKEFUN_INPUT_DIR` (else `/input`) and writes
`$MAKEFUN_OUTPUT_DIR/<id>.out` with 5 quantiles (0.05, 0.25, 0.5, 0.75,
0.95) for each of the next 240 minutes, per market.

## Model

- **Spread scale (per market):** realized volatility of 1-minute returns
  over the last 360 minutes, log-shrunk 0.7 toward the pooled volatility
  across markets, x0.85, floored at 0.25x pooled, capped at 100x pooled.
- **Spread shape:** empirical quantiles of h-minute price changes over
  the last 720 minutes, standardized by each market's scale, shrunk 0.6
  toward normal z-quantiles. This fits the flat-mostly / occasional-jump
  structure of prediction-market prices better than a fixed normal spread.
- **Median:** last observed price (no drift — drift variants were tested
  and rejected: all worse on both tune and holdout).
- Quantiles clamped to [0,1], rounded to 6 decimals, rows forced
  non-decreasing. Fully deterministic: two runs on the same input are
  byte-identical (verified on p00).

## Practice scores (pinball loss, lower is better)

| case | loss |
|------|------|
| p00  | 0.4382 |
| p01  | 0.3165 |
| p02  | 0.3667 |
| p03  | 0.2606 |
| p04  | 0.2093 |
| p05  | 0.1024 |
| **total** | **0.271738** |
| tune (p00–p02) | 0.373807 |
| holdout (p03–p05) | 0.190780 |

House persistence baseline (starter, fixed sigma 0.0025): **0.363589**.
Margin: **0.091851 points lower (25.3% better)**, winning on both the
tune half and the holdout half. 109 candidates were tested; this was the
best on total loss while also winning both halves (nearest rivals:
emp720-sh0.3 at 0.272825, emp1080-sh0.6 at 0.274416).

Runtime: ~1 s per case on an idle machine (cap 8 s per case).
Output: 19–24 markets x 240 rows x 5 quantiles, ~170–250 KB per case
(cap 1 MiB).

## Warning

Practice scores do NOT predict the final score: the final is 12 cases
recorded after the deadline from unseen prices. The model was tuned on
p00–p02 and validated on p03–p05 only; no final data was read.

## Run

```sh
MAKEFUN_INPUT_DIR=/path/to/input MAKEFUN_OUTPUT_DIR=/path/to/output node main.ts
```
