# 📐 Black-Scholes Options Pricer

> **Live demo → [benjadeville.github.io/black-scholes](https://benjadeville.github.io/black-scholes)**  
> **Notebook → `black_scholes.ipynb`**

A full implementation of the Black-Scholes model built for practical use by options traders — not just academia. Covers pricing, all five Greeks, implied volatility solving, P&L simulation, and an interactive dashboard where you plug in your own parameters and watch everything update in real time.

---

## Why Black-Scholes still matters in 2026

The model's assumptions are wrong. Volatility is not constant. Returns are not normally distributed. Markets are not continuous.

And yet every options trader on every desk in the world still uses it — because it provides a **universal language**. When a trader says "I'm buying vol at 28", they mean: the market is pricing that option as if Black-Scholes with σ=28% is correct. The model is the benchmark everything else is measured against.

Understanding Black-Scholes deeply — not just the formula, but the mechanics — is the foundation of all derivatives work.

---

## What this project builds

### 1. Core Pricer
European call and put pricing from first principles. Clean Python class, no black boxes.

```python
bs = BlackScholes(S=562, K=562, T=30/365, r=0.043, sigma=0.28)
bs.summary('call')

# OUTPUT:
# ═══════════════════════════════════════════════
#   BLACK-SCHOLES · CALL OPTION PRICER
# ═══════════════════════════════════════════════
#   Spot     :     562.00
#   Strike   :     562.00
#   Expiry   :       30 days
#   Vol      :      28.0%
#   Rate     :       4.3%
# ───────────────────────────────────────────────
#   PRICE    :      18.97
# ───────────────────────────────────────────────
#   Delta    :      0.5335   (hedge ratio)
#   Gamma    :      0.0088   (delta change / $1)
#   Vega     :      0.6405   (P&L / 1% vol move)
#   Theta    :     -0.3320   (P&L / day)
#   Rho      :      0.2309   (P&L / 1% rate move)
# ═══════════════════════════════════════════════
```

### 2. The Five Greeks — Trader Interpretation

| Greek | Formula | What it tells you |
|---|---|---|
| **Delta Δ** | N(d1) | How many shares to hold for a delta-neutral hedge. ATM call ≈ 0.50. |
| **Gamma Γ** | N'(d1) / (S·σ·√T) | How fast your hedge goes stale. Peaks at the forward, not the strike — because Black-Scholes prices on the forward price S·e^(rT), not spot. |
| **Vega ν** | S·N'(d1)·√T / 100 | P&L for a 1% move in implied vol. The most important Greek for vol traders. |
| **Theta Θ** | (complex) / 365 | Daily time decay. Always negative for long options. Accelerates non-linearly toward expiry — the last 7 days are brutal. |
| **Rho ρ** | K·T·e^(-rT)·N(d2) / 100 | Sensitivity to interest rates. Matters most on long-dated options. |

**Note on Gamma peak location:** The gamma peak is not at the strike K — it is at the **forward price F = S·e^(rT)**. With S=562, r=4.3%, T=30d, the forward is F≈$564. This is where delta uncertainty is maximized. On long-dated options (6M, 1Y), this shift becomes significant and traders manage it explicitly as "forward delta" vs "spot delta".

### 3. Implied Volatility Solver

Black-Scholes takes vol as input and gives price as output. The IV solver does the reverse — given a market price, find the σ that makes the model match. Implemented via Brent's method (more robust than Newton-Raphson near expiry where vega flattens).

```python
# Market is quoting $20.50 — what vol does that imply?
iv = implied_vol(market_price=20.50, S=562, K=562, T=30/365, r=0.043)
# → 30.39% — market pricing +2.39% more uncertainty than our model
```

This is how traders actually use the model: not to get the "right" price, but to detect **vol discrepancies**. If you think realized vol will be 28% and the market implies 30.39%, you sell the option and collect the premium.

### 4. P&L Simulation

Three-panel visualization for any position (long/short call/put, any number of contracts):
- **Payoff diagram** — P&L at expiry vs today across ±20% spot range
- **Theta decay** — ATM vs ITM vs OTM decay curves (note the non-linear acceleration)
- **P&L heatmap** — full grid of spot × time remaining

### 5. Interactive Dashboard

Plug in any parameters (S, K, T, σ, r) and watch all charts update live. Built for traders who want to stress-test a position before entering — change vol from 20% to 40%, move the strike, compress time to expiry.

---

## The Formula

```
Call = S·N(d1) - K·e^(-rT)·N(d2)
Put  = K·e^(-rT)·N(-d2) - S·N(-d1)

d1 = [ln(S/K) + (r + σ²/2)·T] / (σ·√T)
d2 = d1 - σ·√T
```

Where N(x) is the cumulative standard normal distribution.

**Intuition:**
- N(d2) = risk-neutral probability the option expires ITM
- N(d1) = delta of the call (also related to the probability of exercise under the stock measure)
- The formula = expected value of receiving the stock minus expected cost of paying the strike

**Why call > put for ATM options?**
Put-call parity: `C - P = S - K·e^(-rT)`. With r > 0, the right-hand side is positive even when S = K. The call is worth more because the buyer has use of the cash (invested at r) while waiting — the put seller has the opposite carry.

---

## Repo Structure

```
black-scholes/
├── black_scholes.ipynb     # Full notebook — theory + code + charts
├── black_scholes.py        # Clean Python module (importable)
├── index.html              # Interactive trader dashboard (GitHub Pages)
├── pnl_simulation.png      # Generated chart
├── greeks_profile.png      # Generated chart
└── README.md
```

---

## Quickstart

```bash
git clone https://github.com/Benjadeville/black-scholes.git
cd black-scholes
pip install numpy scipy matplotlib
jupyter notebook black_scholes.ipynb
```

---

## Limitations and extensions

| Limitation | Extension |
|---|---|
| Constant volatility | Local vol (Dupire), stochastic vol (Heston) |
| No dividends | Add continuous dividend yield q → replace S with S·e^(-qT) |
| European only | Binomial tree or LSM Monte Carlo for American options |
| Log-normal returns | Jump-diffusion (Merton) for fat tails |
| Single underlying | Correlation models (basket options) |

---

## Related Projects

- [`gex-dealer`](https://github.com/Benjadeville/gex-dealer) — Dealer gamma exposure, gamma flip level — built on top of these Greeks
- [`cta-positioning`](https://github.com/Benjadeville/cta-positioning) — CTA systematic exposure model
- [`etf-short-monitor`](https://github.com/Benjadeville/etf-short-monitor) — ETF short interest and squeeze detection

---

*Not financial advice. Educational project.*