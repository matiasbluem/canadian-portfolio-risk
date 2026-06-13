# Canadian Portfolio Risk Analytics

Quantitative risk analysis of a 10-asset Canadian portfolio using 10 years of market data. Computes VaR, max drawdown, Monte Carlo simulation, and MPT optimization, then outputs an
interactive browser-based dashboard.

**Live dashboard:** [View here](https://matiasbluem.github.io/canadian-portfolio-risk/portfolio_dashboard.html)

---

## About

Built as a portfolio project to demonstrate quantitative finance and data analytics skills. Context: Business Management student at Toronto Metropolitan University targeting analytics 
and finance roles in the GTA.

---

## What this project does

Takes 10 Canadian assets (5 ETFs + 5 stocks), pulls 10 years of adjusted close prices from Yahoo Finance, and runs the following:

- **Risk metrics** -- annualized return, volatility, Sharpe ratio, max drawdown, VaR at 95% and 99% confidence (historical method)
- **Monte Carlo simulation** -- 1,000 portfolio paths over 252 trading days to quantify 1-year downside risk
- **Portfolio optimization** -- 10,000 random weight combinations to map the efficient frontier and identify the max-Sharpe and min-volatility allocations
- **Stress testing** -- three scenarios: market crash (-20%), doubled volatility, and a rate shock (bonds -10%, equities -5%)

---

## Asset universe

| Ticker | Name | Type |
|--------|------|------|
| XIC.TO | iShares S&P/TSX Composite | ETF |
| XFN.TO | iShares Financials | ETF |
| XEG.TO | iShares Energy | ETF |
| ZAG.TO | BMO Aggregate Bond | ETF |
| XIU.TO | iShares S&P/TSX 60 | ETF |
| RY.TO | Royal Bank of Canada | Stock |
| TD.TO | TD Bank | Stock |
| BNS.TO | Bank of Nova Scotia | Stock |
| CNR.TO | Canadian National Railway | Stock |
| SU.TO | Suncor Energy | Stock |

Bond ETF (ZAG) is included because it has low correlation with equities, which changes the efficient frontier and cushions drawdowns in stress scenarios.

---

## Project structure

```
canadian-portfolio-risk/
|
|-- Canadian_Portfolio_Risk_Analytics.ipynb   # Main notebook (run in Google Colab)
|-- portfolio_dashboard.html                  # Self-contained interactive dashboard
|-- requirements.txt                          # Python dependencies
|-- outputs/                                  # CSVs exported for Tableau
|   |-- tableau_prices.csv
|   |-- tableau_returns.csv
|   |-- tableau_risk_metrics.csv
|   |-- tableau_optimal_weights.csv
|   |-- tableau_drawdowns.csv
|   |-- tableau_scenarios.csv
|   |-- phase4_monte_carlo_percentiles.csv
|   |-- phase5_all_portfolios.csv
|-- README.md
```

---

## How to run

**Option 1 -- Google Colab (recommended)**

1. Open `Canadian_Portfolio_Risk_Analytics.ipynb` in [Google Colab](https://colab.research.google.com)
2. Run cells top to bottom with `Shift + Enter`
3. The first cell installs `yfinance` automatically
4. All outputs save to an `outputs/` folder; the last cell downloads them as a ZIP

**Option 2 -- Local**

```bash
git clone https://github.com/matiasbluem/canadian-portfolio-risk.git
cd canadian-portfolio-risk
pip install -r requirements.txt
jupyter notebook Canadian_Portfolio_Risk_Analytics.ipynb
```

---

## Key findings

These are based on Jan 2014 to Jun 2026 data. Your run will reflect current market data.

**Optimization**
- The max-Sharpe portfolio is heavily concentrated in CNR and RY, which have the best return-per-unit-of-risk in the universe. Equal weighting leaves Sharpe ratio on the table.
- ZAG (bonds) gets meaningful weight in the min-volatility portfolio but near zero in max-Sharpe -- its low return is a drag on Sharpe even though it reduces portfolio vol.

**Diversification**
- XEG and SU have the highest volatility (26-27%) and the lowest Sharpe ratios. Energy sector exposure adds tail risk without proportionate return over this period.
- ZAG correlation with equities is near zero or slightly negative, confirming its role as a diversifier. This is most visible in the March 2020 drawdown, where ZAG held flat while equities dropped 30%+.

**Downside risk**
- The Monte Carlo median after 1 year is close to the annualized return estimate, but the P5 scenario (worst 5% of paths) shows meaningful capital loss even with an optimized portfolio. Risk does not disappear with optimization -- it gets rebalanced.
- The market crash stress test (-20% shock) flips the portfolio from positive to negative annual return and pushes max drawdown past -45%. VaR 99% stays relatively stable because it is calculated from historical daily returns, not the shock itself.

---

## Methods

**Value at Risk (historical method)**
Takes the 5th and 1st percentile of actual historical daily returns. No distributional assumption. The limitation is that it assumes the future looks like the past -- which fails during regime changes, which is why the stress tests exist alongside VaR.

**Monte Carlo simulation**
Draws random daily returns from a normal distribution parameterized by the portfolio's historical mean and standard deviation, compounds them over 252 days, and repeats 1,000 times. Key assumption: normally distributed returns. Real return distributions have fatter tails, so the worst-case scenarios in practice can exceed what the simulation shows.

**Modern Portfolio Theory**
Samples 10,000 random weight vectors (Dirichlet distribution, so weights sum to 1), computes annualized return and volatility for each using the historical covariance matrix, and selects the portfolio with the highest Sharpe ratio. This is a random search, not a convex optimizer -- results vary slightly across runs. Setting `np.random.seed(0)` makes them reproducible.

**Stress testing**
Applies one-time return shocks to the historical return series and recalculates all metrics. The crash scenario adds a single -20% return day to all equity positions. The doubled-volatility scenario scales daily returns around their mean by a factor of 2. The rate shock applies -10% to ZAG and -5% to all equities simultaneously.

---

## Tech stack

| Tool | Purpose |
|------|---------|
| Python 3.10+ | All analysis |
| yfinance | Price data download |
| pandas | Data cleaning and manipulation |
| NumPy | Return and risk calculations |
| SciPy | KDE for VaR distribution chart |
| Matplotlib | Charts inside the notebook |
| Chart.js | Interactive HTML dashboard |

---

## Dashboard

`portfolio_dashboard.html` is self-contained -- no server required. Open it in any browser. It includes:

- Normalized price history (all 10 assets)
- Risk vs. return scatter colored by Sharpe ratio
- Efficient frontier with 4,000 portfolios plotted
- Drawdown chart for the XIC benchmark
- Portfolio allocation switcher (Max Sharpe / Min Vol / Equal Weight)
- Monte Carlo fan chart (P5, P25, P50, P75, P95)
- Stress test scenario cards

