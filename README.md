# Macroeconomic Regime‑Switching Model

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A complete implementation of a **Markov switching model** to identify economic regimes (e.g., expansion/recession) from quarterly GDP growth, and a simple asset allocation strategy that switches between stocks (SPY) and bonds (IEF) based on the inferred regime probability. This project demonstrates how macroeconomic signals can be used for dynamic portfolio management.

---

## Overview

Economic conditions are not constant; they alternate between periods of expansion and recession, each with distinct characteristics. This project:

- Fetches quarterly US GDP data from FRED.
- Fits a two‑regime Markov switching model with switching mean and variance.
- Uses **filtered probabilities** (real‑time, no look‑ahead) to determine the current regime.
- Automatically identifies the “good” regime (higher mean growth) as the expansion state.
- Backtests a simple strategy: allocate 100% to stocks (SPY) when the probability of being in the good regime exceeds 50%, otherwise allocate to bonds (IEF).
- Compares the strategy against buy‑and‑hold 60/40, 100% stocks, and 100% bonds.

The key insight is that **filtered probabilities** are essential for realistic backtesting – they use only information available up to that point in time, unlike smoothed probabilities which incorporate future data.

---

## Data Sources

- **GDP Growth**: `GDPC1` (Real Gross Domestic Product) from FRED (quarterly, 1960–2023).  
  Retrieved via `pandas_datareader`.
- **Asset Prices**: SPY (S&P 500 ETF) and IEF (7‑10 Year Treasury ETF) from Yahoo Finance (monthly, 2002–2024).  
  Retrieved via `yfinance`.

---

## Methodology

### 1. Markov Switching Model

We model quarterly GDP growth as a two‑regime process:
y_t = μ_{s_t} + ε_t, ε_t ~ N(0, σ_{s_t}^2)

where `s_t ∈ {0,1}` is an unobserved regime. Both the mean and variance are allowed to switch. The regimes evolve according to a Markov chain with transition probabilities `p_{ij}`.

### 2. Estimation

The model is estimated using maximum likelihood via the Hamilton filter (implemented in `statsmodels`). The output includes:

- Regime‑specific means and variances.
- Transition probabilities.
- **Filtered probabilities**: `P(s_t = i | y_1, …, y_t)` – the best real‑time estimate of the current regime.
- **Smoothed probabilities**: `P(s_t = i | y_1, …, y_T)` – retrospective estimates using all data.

### 3. Identifying the “Good” Regime

We compare the two estimated means:

- **Regime 0**: mean = 0.76%, volatility = 0.47%
- **Regime 1**: mean = 0.71%, volatility = 1.74%

Regime 0 has a higher mean and lower volatility → it is identified as the **expansion regime**. The filtered probability of this regime is used for trading decisions.

### 4. Asset Allocation Strategy

- Monthly rebalancing.
- At the beginning of each month, we use the most recent filtered probability of the expansion regime (lagged by one month to account for GDP data release delay).
- If probability > 0.5 → invest 100% in SPY.
- Else → invest 100% in IEF.
- No transaction costs (for simplicity; can be added).

### 5. Benchmarks

- **60/40**: 60% SPY, 40% IEF rebalanced monthly.
- **SPY only**: 100% stocks.
- **IEF only**: 100% bonds.

---

## Results

**Backtest Performance (2002–2024)**

| Strategy         | Annual Return | Annual Volatility | Sharpe Ratio | Max Drawdown |
|------------------|---------------|-------------------|--------------|--------------|
| Regime‑Switch    | 10.57%        | 12.43%            | 0.85         | 177.91%      |
| 60/40            | 8.14%         | 9.03%             | 0.90         | 106.44%      |
| SPY (stocks)     | 10.73%        | 14.83%            | 0.72         | 181.75%      |
| IEF (bonds)      | 3.46%         | 6.77%             | 0.51         | 59.81%       |

### Interpretation

- The regime‑switching strategy achieved an **annual return of 10.57%**, nearly matching the pure equity return (10.73%) but with **lower volatility** (12.43% vs 14.83%). This results in a higher Sharpe ratio than SPY (0.85 vs 0.72).
- Compared to a simple 60/40 portfolio, the regime strategy delivered higher return but also higher volatility and drawdown; its Sharpe is slightly lower (0.85 vs 0.90). The 60/40 remains a tough benchmark.
- The maximum drawdown of 177.91% is still substantial, indicating that the strategy remained in equities during some severe downturns (e.g., 2008 financial crisis, 2020 COVID crash) when the filtered probability of expansion might have dropped too late. This suggests that a smoother allocation (e.g., scaling positions with probability) or a more responsive indicator could further reduce drawdowns.
- Importantly, the use of **filtered probabilities** (instead of smoothed) eliminates look‑ahead bias and provides a realistic assessment of what could have been achieved in real time.

---
<img width="999" height="468" alt="image" src="https://github.com/user-attachments/assets/acd76e43-2fee-4f23-a48a-03e87b8cd589" />
<img width="1001" height="545" alt="image" src="https://github.com/user-attachments/assets/b2363713-8e50-4474-ac9e-e2d8a45c689b" />
<img width="1389" height="590" alt="image" src="https://github.com/user-attachments/assets/ebb72d5c-35eb-456a-943e-1b06e7640d7c" />
<img width="997" height="545" alt="image" src="https://github.com/user-attachments/assets/2e0c4b96-8b7e-40cd-a5c7-4d3f94ebbb58" />

