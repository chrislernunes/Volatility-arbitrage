# BankNifty Dispersion Trading Strategy

Volatility arbitrage backtest on BankNifty and its constituents (Jul–Dec 2017).

---

## Overview

Dispersion trading exploits the structural premium in index implied volatility relative to the weighted IV of its constituent stocks. The strategy profits from mean-reversion of implied correlation.

- **Index:** BANKNIFTY
- **Constituents:** AXISBANK, HDFCBANK, ICICIBANK, KOTAKBANK, SBIN
- **Period:** July – December 2017
- **Risk-Free Rate:** 6% (RBI repo rate)

---

## Core Signal

Dirty implied correlation is defined as:

```
rho_dirty = ( IV_BankNifty / sum(w_i * IV_i) )^2
```

A 20-day rolling z-score of `rho_dirty` drives all trading decisions:

| Z-Score   | Condition            | Trade                          |
|-----------|----------------------|--------------------------------|
| > +0.5    | Index IV expensive   | Short index + Long stocks      |
| < -0.5    | Index IV cheap       | Long index + Short stocks      |
| → 0.0     | Mean reversion       | Exit all positions             |

---

## IV Engine

Custom vectorized Black-Scholes solver replacing `mibian`. Runs ~400,000 options/sec (10–50x faster).

1. Corrado-Miller rational approximation for initial IV guess
2. Vectorized Newton-Raphson using vega as the derivative
3. Brent's method fallback for the <1% of options that don't converge
4. Intrinsic value filter + round-trip sanity check on startup

---

## Position Sizing

NSE F&O official lot sizes (2017), scaled by index weight:

```
Symbol        Lot Size    Weight    Lot x Weight
----------    --------    ------    ------------
HDFCBANK           500     0.333           166.5
ICICIBANK        2,500     0.173           432.5
KOTAKBANK          800     0.123            98.4
SBIN             3,000     0.102           306.0
AXISBANK         1,200     0.080            96.0
BANKNIFTY           40     1.000            40.0
```

Constituent legs take the **opposite sign** to the index leg. Weights are not renormalized.

---

## P&L Methodology

Daily P&L uses a vega-based approximation:

```
leg_pnl = position x dIV x vega_per_share x lot_size x weight
```

ATM straddle vega approximation:

```
vega_per_share = 2 * S * sqrt(T) / sqrt(2*pi)
```

---

## Data Pipeline

1. Load `BankNifty_OP_Data.csv` — daily option chain snapshots
2. Price = LTP → Close → Settle Price (best available)
3. Drop zero-price and zero-spot rows
4. Deduplicate — keep highest open-interest row per (date, symbol, strike, type)
5. Keep front-month expiry only per (date, symbol)
6. ATM strike = closest to futures price; average call + put IV per day

---

## Delta Hedging

```
net_delta_shares = pos * delta_BN * lot_BN
                 + sum( -pos * delta_i * lot_i * w_i )

hedge_contracts  = -net_delta_shares / lot_BN
```

Hedge is expressed in BankNifty futures contracts.

---

## Performance Metrics

| Metric         | Definition                                        |
|----------------|---------------------------------------------------|
| Sharpe Ratio   | (mean_ret − rf/252) / std_ret × √252              |
| Calmar Ratio   | Annualised return / \|max drawdown %\|            |
| Win Rate       | Profitable days / active trading days             |
| Profit Factor  | \|avg win\| / \|avg loss\|                        |
| Max Drawdown   | Peak-to-trough cumulative P&L decline             |

---

## Visualizations

- ATM implied volatility over time — one panel per symbol (3×2 grid)
- Dirty correlation + z-score with shaded signal regions
- Cumulative P&L, drawdown, and daily P&L bars (3-panel)
- BankNifty IV vs weighted constituent IV and spread
- Monthly P&L bar chart with annotated values
- Pairwise ATM IV correlation heatmap

---

## Key Parameters

```
ENTRY_Z   = 0.5       # z-score entry threshold
EXIT_Z    = 0.0       # z-score exit threshold
ROLL_WIN  = 20        # rolling window in days
RISK_FREE = 0.06      # risk-free rate
REF_NOT   = 960,000   # 1 BN lot x ~24,000 spot (for % return display)
```

---

## Requirements

```
numpy
pandas
scipy
matplotlib
seaborn
```
