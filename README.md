# FX-Carry-Trade-Currency-Risk-Analytics-Engine
from pathlib import Path

readme = # FX Carry Trade & Currency Risk Analytics Engine

A systematic quantitative FX strategy that tests whether cross-sectional interest-rate differentials can generate risk-adjusted returns after incorporating exchange-rate movements, transaction costs, volatility targeting, and tail-risk analysis.

## Project Overview

This project implements a **monthly FX carry trade strategy** across six major currencies relative to USD:

- EUR — Euro
- GBP — British Pound
- AUD — Australian Dollar
- JPY — Japanese Yen
- CAD — Canadian Dollar
- CHF — Swiss Franc

The strategy ranks currencies by their interest-rate differential versus USD, goes long the highest-carry currencies, shorts the lowest-carry currencies, and evaluates the resulting portfolio using quantitative performance and risk metrics.

The project is designed as a practical quantitative-finance research pipeline rather than a simple backtest. It includes data engineering, signal construction, portfolio formation, transaction-cost modeling, volatility targeting, benchmark comparison, econometric testing, and stress-period analysis.

---

## Strategy Logic

The baseline strategy follows these steps:

1. Collect monthly FX prices and interest-rate data.
2. Normalize FX quote conventions so currency returns have a consistent interpretation.
3. Calculate each currency's interest-rate differential relative to USD.
4. Rank currencies cross-sectionally by carry.
5. **Long the two highest-carry currencies.**
6. **Short the two lowest-carry currencies.**
7. Allocate equal weights:
   - +50% to each long position
   - -50% to each short position
8. Combine FX price returns and carry returns.
9. Deduct estimated transaction costs based on portfolio turnover.
10. Apply volatility targeting with a 10% annualized target.
11. Compare strategy performance with an equal-weight FX benchmark.
12. Evaluate performance, downside risk, tail risk, and behavior during stress periods.

---

## Key Configuration

| Parameter | Setting |
|---|---|
| Base / funding currency | USD |
| FX universe | EUR, GBP, AUD, JPY, CAD, CHF |
| Rebalancing frequency | Monthly |
| Carry ranking | Cross-sectional |
| Long positions | Top 2 currencies |
| Short positions | Bottom 2 currencies |
| Long weight | +50% each |
| Short weight | -50% each |
| Target volatility | 10% annualized |
| Volatility lookback | 12 months |
| Transaction cost | 10 bps |
| Sharpe risk-free rate | 0% |
| FX data | Yahoo Finance |
| Interest-rate data | FRED |

---

## Quantitative Framework

### 1. Carry Signal

For currency $i$, the carry signal is approximated as:

$$ \text{Carry}_{i,t} = r^{\text{foreign}}_{i,t} - r^{\text{USD}}_{t} $$

where:
* $r^{\text{foreign}}_{i,t} = \text{interest rate associated with currency } i$
* $r^{\text{USD}}_{t} = \text{USD interest rate}$

A higher positive differential indicates greater theoretical carry from holding the foreign currency relative to USD.[cite: 1]

### 2. FX Return

After normalizing quote conventions:

$$ R^{\text{FX}}_{i,t} = \frac{P_{i,t}}{P_{i,t-1}} - 1 $$

The normalization step is important because FX pairs can be quoted in opposite directions. For example, USD/JPY must be handled consistently with EUR/USD-type quotations before calculating portfolio returns[cite: 3].
### 3. Carry Return

The notebook uses a simplified monthly approximation:

$$
R^{Carry}_{i,t}
\approx
\frac{\text{Carry}_{i,t}}{12}
$$

### 4. Total Currency Return

$$ R^{\text{Total}}_{i,t} = R^{\text{FX}}_{i,t} + R^{\text{Carry}}_{i,t} $$

### 5. Portfolio Return

The strategy combines currency returns according to the monthly long/short weights:

$$ R_{p,t} = \sum_i w_{i,t} R^{\text{Total}}_{i,t} $$

Transaction costs are then deducted:

$$ R^{\text{Net}}_{p,t} = R_{p,t} - TC_t $$

where transaction costs depend on portfolio turnover[cite: 5].

### 6. Volatility Targeting

The strategy scales exposure toward a 10% annualized volatility target using a 12-month volatility estimate[cite: 6].

Conceptually:

$$ \text{Leverage}_{t} = \frac{\sigma_{\text{target}}}{\sigma_{\text{estimated},t}} $$
> **Important implementation note:** volatility targeting should use a lagged volatility estimate when used for a strict out-of-sample backtest. For example, the rolling volatility estimate should be shifted by one period before determining the next month's position size. This prevents look-ahead bias.

---

## Data Pipeline

### Market Data

FX prices are sourced from **Yahoo Finance**.

The pipeline:

1. Downloads FX time series.
2. Handles currency-pair quote conventions.
3. Converts observations to monthly frequency.
4. Calculates monthly FX returns.
5. Aligns the FX data with interest-rate observations.

### Interest-Rate Data

Interest-rate information is sourced from **FRED**.

The pipeline:

1. Retrieves relevant interest-rate series.
2. Aligns observations to the monthly strategy frequency.
3. Handles missing observations.
4. Calculates currency-versus-USD rate differentials.
5. Uses these differentials as the carry signal.

---

## Portfolio Construction

At every monthly rebalance:

```text
                Interest-rate differential
                           │
                           ▼
                    Rank currencies
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        Highest carry              Lowest carry
              │                         │
              ▼                         ▼
        Long top 2                 Short bottom 2
        +50% each                  -50% each
              └────────────┬────────────┘
                           ▼
                    Portfolio returns
                           │
                           ▼
                 Transaction costs
                           │
                           ▼
                 Volatility targeting
                           │
                           ▼
                    Net strategy return
```

The portfolio is therefore a **dollar-neutral long/short FX carry strategy** in terms of the initial equal-weight construction.

---

## Risk & Performance Analysis

The project evaluates the strategy using:

- CAGR
- Annualized volatility
- Sharpe ratio
- Sortino ratio
- Calmar ratio
- Maximum drawdown
- Skewness
- Value at Risk (VaR)
- Expected Shortfall / CVaR
- Worst monthly return
- Turnover
- Rolling Sharpe ratio

These metrics are intended to evaluate both return generation and the quality of the return distribution.

### Why these metrics matter

**Sharpe ratio**  
Measures return relative to total volatility.

**Sortino ratio**  
Focuses more specifically on downside volatility.

**Calmar ratio**  
Compares annualized return with maximum drawdown.

**Maximum drawdown**  
Measures the largest peak-to-trough portfolio decline.

**VaR**  
Estimates a return-loss threshold at a specified tail percentile.

**Expected Shortfall / CVaR**  
Measures the average loss beyond the VaR threshold and therefore provides additional information about tail severity.

**Skewness**  
Helps identify whether returns are asymmetric, which is particularly relevant for carry strategies because carry trades can experience sharp crash-like losses.

---

## Visualizations

The project produces several research-oriented visualizations.

### Cumulative Performance

Shows how the strategy's value evolves over time and allows comparison with the benchmark.

### Drawdown

Shows periods when the strategy falls below previous portfolio peaks.

This is particularly important for identifying prolonged or severe losses.

### Carry Spreads

Shows the cross-sectional interest-rate differentials that drive the trading signal.

### Rolling Sharpe Ratio

Shows whether risk-adjusted performance is stable or highly regime-dependent.

### Return Distribution

Displays the distribution of monthly strategy returns and helps identify asymmetry and tail events.

### Currency Performance

Shows how individual currencies contribute to overall strategy behavior.

### Stress-Period Analysis

The project evaluates behavior during major market regimes, including:

- Global Financial Crisis
- COVID-19 period
- 2022 rate/risk regime

This is important because FX carry strategies can perform well during stable risk-on environments but experience sharp losses during volatility shocks and funding stress.

---

## Econometric Test

The project also tests the relationship between carry and subsequent FX returns.

The basic specification is:

\[
R^{FX}_{i,t+1}
=
\alpha
+
\beta Carry_{i,t}
+
\epsilon_{i,t+1}
\]

The objective is to examine whether currencies with higher current carry subsequently exhibit different FX returns.

Heteroskedasticity and autocorrelation-consistent (HAC) standard errors are used in the regression analysis.

This provides an additional statistical perspective beyond portfolio-level backtesting.

---

## Transaction Costs

The baseline implementation assumes a constant transaction cost of:

**10 basis points (0.10%)**

Transaction costs are linked to portfolio turnover.

A simplified turnover measure is based on absolute changes in portfolio weights:

\[
Turnover_t =
\sum_i |w_{i,t}-w_{i,t-1}|
\]

Estimated trading costs are then deducted from gross portfolio returns.

---

## Benchmark

The project uses an **equal-weight FX basket** as a benchmark.

The benchmark provides a reference point for determining whether the carry strategy generates performance beyond a simple diversified FX allocation.

The benchmark should not be interpreted as a pure factor benchmark for carry. Additional factor benchmarks would strengthen the analysis.

---

## Project Architecture

```text
FXCarryTrade_CurrencyRisk_AnalyticsEngine.ipynb
│
├── Configuration
│   ├── Currency universe
│   ├── Strategy parameters
│   └── Risk parameters
│
├── Data Collection
│   ├── Yahoo Finance FX prices
│   └── FRED interest-rate data
│
├── Data Cleaning
│   ├── Frequency alignment
│   ├── Missing-value handling
│   └── FX quote normalization
│
├── Feature Engineering
│   ├── FX returns
│   ├── Interest-rate differentials
│   ├── Carry returns
│   └── Rolling risk measures
│
├── Signal Generation
│   └── Cross-sectional carry ranking
│
├── Portfolio Construction
│   ├── Long top 2
│   ├── Short bottom 2
│   └── Equal weighting
│
├── Backtesting
│   ├── Gross returns
│   ├── Turnover
│   ├── Transaction costs
│   └── Volatility targeting
│
├── Risk Analytics
│   ├── Sharpe
│   ├── Sortino
│   ├── Drawdown
│   ├── VaR
│   └── Expected Shortfall
│
├── Statistical Analysis
│   └── Carry-return regression
│
└── Stress Testing
    ├── GFC
    ├── COVID-19
    └── 2022 regime
```

---

## Technology Stack

- **Python**
- **Pandas** — data manipulation and time-series processing
- **NumPy** — numerical calculations
- **Matplotlib** — visualization
- **Statsmodels** — econometric analysis
- **SciPy** — statistical calculations
- **Yahoo Finance** — FX market data
- **FRED** — interest-rate data
- **Google Colab** — development environment

---

## Project Structure

A recommended GitHub repository structure is:

```text
FX-Carry-Trade-Currency-Risk-Analytics/
│
├── FXCarryTrade_CurrencyRisk_AnalyticsEngine.ipynb
├── README.md
├── requirements.txt
│
├── data/
│   ├── raw/
│   └── processed/
│
├── charts/
│   ├── cumulative_performance.png
│   ├── drawdown.png
│   ├── carry_spreads.png
│   ├── rolling_sharpe.png
│   └── return_distribution.png
│
├── outputs/
│   └── strategy_results.csv
│
└── reports/
    └── FX_Carry_Trade_Project_Report.pdf
```

---

## Installation

Install the main Python dependencies with:

```bash
pip install pandas numpy matplotlib scipy statsmodels yfinance
```

If using Google Colab:

```python
!pip install yfinance statsmodels
```

---

## How to Run

### Option 1 — Google Colab

1. Open the notebook in Google Colab.
2. Run the setup/import cells.
3. Run the data-download cells.
4. Execute the data-cleaning and feature-engineering sections.
5. Run the carry signal and portfolio-construction cells.
6. Execute the backtest.
7. Review performance, risk, regression, and stress-test outputs.

### Option 2 — Local Jupyter Environment

Clone the repository:

```bash
git clone <your-repository-url>
cd FX-Carry-Trade-Currency-Risk-Analytics
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Launch Jupyter:

```bash
jupyter notebook
```

Then open:

```text
FXCarryTrade_CurrencyRisk_AnalyticsEngine.ipynb
```

---

## Important Assumptions & Limitations

This is a research and educational backtesting framework, not a production trading system.

### 1. Simplified Carry Calculation

The project approximates carry as:

\[
\frac{Foreign\ Rate-USD\ Rate}{12}
\]

A production FX strategy would ideally use actual forward points, FX swaps, funding curves, cross-currency basis, and realistic roll conventions.

### 2. Simplified Transaction Costs

A constant 10 bps assumption does not capture:

- Bid-ask spreads
- Market depth
- Currency-specific liquidity
- Trade size
- Volatility-dependent market impact
- Execution timing

### 3. Rate Data Timing

Blind forward-filling of interest-rate observations can potentially introduce stale information.

A production implementation should account for publication dates and limit how long observations can be carried forward.

### 4. Volatility-Targeting Timing

The volatility estimate used for position sizing must be available before the return being traded.

Using contemporaneous information can create look-ahead bias.

### 5. Benchmark Choice

The equal-weight FX basket is useful as a broad reference, but it is not a pure carry-factor benchmark.

### 6. Historical Backtest Risk

Past performance does not guarantee future performance. FX strategies can be highly regime-dependent.

---

## Recommended Upgrades

### Upgrade 1 — Remove Look-Ahead Bias

Use a lagged rolling volatility estimate:

```python
rolling_vol = (
    returns.rolling(12).std()
    * np.sqrt(12)
).shift(1)
```

Then use this estimate to determine the next period's position size.

### Upgrade 2 — Use FX Forward Points

Replace the simplified interest-rate carry approximation with tradable forward pricing.

This would make the strategy much closer to how institutional FX carry strategies are actually implemented.

### Upgrade 3 — Add Walk-Forward Testing

Separate:

```text
Training / Research Period
          ↓
Parameter Selection
          ↓
Out-of-Sample Period
          ↓
Performance Evaluation
```

This reduces the risk of overfitting the strategy to historical data.

### Upgrade 4 — Improve Execution Modeling

Use:

- Currency-specific spreads
- Liquidity-adjusted costs
- Position limits
- Market-impact assumptions
- Slippage
- Rebalancing constraints

### Upgrade 5 — Add Additional FX Signals

Compare:

```text
Carry only
Carry + Momentum
Carry + Value
Carry + Momentum + Value
```

This would allow analysis of whether carry provides independent alpha or whether performance is explained by other currency factors.

---

## What This Project Demonstrates

This project demonstrates practical experience with:

- Quantitative strategy design
- FX market data
- Interest-rate differentials
- Cross-sectional ranking
- Long/short portfolio construction
- Time-series analysis
- Transaction-cost modeling
- Volatility targeting
- Risk management
- Tail-risk measurement
- Econometric testing
- Stress testing
- Financial data engineering
- Python-based quantitative research

---

## Quant Finance Interview Summary

A concise way to describe the project in an interview:

> I built a systematic monthly FX carry strategy across six major currencies relative to USD. I normalized FX quote conventions, calculated interest-rate differentials, ranked currencies cross-sectionally, went long the two highest-carry currencies and short the two lowest-carry currencies, and combined FX and carry returns. I then modeled turnover and transaction costs, applied volatility targeting, and evaluated the strategy using Sharpe, Sortino, drawdown, VaR, Expected Shortfall, regression analysis, and stress testing. I also identified look-ahead risk in the original volatility-targeting implementation and proposed a lagged volatility estimate as the correction.

---

## Disclaimer

This project is for **research, educational, and portfolio-development purposes only**.

It does not constitute investment advice, a recommendation to trade currencies, or evidence of guaranteed future performance.

All strategy results depend on the underlying data, assumptions, implementation choices, and historical market conditions.

---

## Author

**Asraf**

Quantitative Finance | Financial Data Analytics | Python

GitHub: `asrafff-25`
"""

path = Path("/mnt/data/README.md")
path.write_text(readme, encoding="utf-8")
print(f"Created: {path}")
