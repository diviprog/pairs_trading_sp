# Pairs Trading Strategy on S&P 500 Stocks

A comprehensive research-quality implementation of statistical arbitrage using cointegration-based pairs trading on S&P 500 stocks.

## Overview

This project implements a pairs trading strategy that identifies cointegrated stock pairs within the S&P 500 and trades the mean-reverting spread between them. The project includes both a baseline implementation and an improved version with multiple research-backed enhancements.

### Key Features

- **Cointegration Testing**: Engle-Granger two-step method for identifying statistically related pairs
- **Walk-Forward Analysis**: Realistic out-of-sample backtesting with rolling windows
- **VIX Regime Detection**: Market regime-aware trading logic
- **Sector-Based Pair Selection**: Within-sector pairs for fundamental similarity
- **Multiple Performance Improvements**: Persistence filtering, half-life analysis, Kelly criterion sizing

## Project Structure

```
pairs_trading_sp/
├── pairs_trading.ipynb           # Baseline implementation
├── pairs_trading_improved.ipynb  # Enhanced strategy with improvements
├── requirements.txt              # Python dependencies
├── data/                         # Generated data files
│   ├── sp500_prices.csv          # Historical price data (2015-2026)
│   ├── sp500_prices_extended.csv # Extended price data (2000-2026)
│   ├── sp500_returns.csv         # Daily returns
│   ├── sp500_info.csv            # Stock metadata
│   ├── vix.csv                   # VIX volatility index
│   ├── cointegrated_pairs.csv    # Identified cointegrated pairs
│   ├── backtest_results.csv      # Individual pair backtest results
│   ├── walkforward_results.csv   # Walk-forward analysis results
│   ├── pair_persistence.csv      # Pair persistence across windows
│   ├── top_pairs.csv             # Best performing pairs
│   ├── ticker_sectors.json       # Ticker to sector mapping
│   └── config_comparison.csv     # Window configuration comparison
└── README.md
```

## Notebooks

### 1. Baseline Implementation (`pairs_trading.ipynb`)

The baseline notebook covers the complete pairs trading pipeline:

| Section | Description |
|---------|-------------|
| Data Collection | Download S&P 500 prices via yfinance |
| Data Cleaning | Handle missing data, filter by quality |
| Cointegration Testing | Engle-Granger test for all within-sector pairs |
| Spread Visualization | Analyze spread behavior and z-scores |
| Trading Logic | Z-score based entry/exit signals |
| Backtesting | Performance metrics for individual pairs |
| Walk-Forward Analysis | Rolling window out-of-sample testing |
| VIX Regime Detection | Crisis vs normal period analysis |
| Parameter Sensitivity | Entry/exit threshold optimization |
| Risk Management | Kelly criterion, VaR, drawdown analysis |

**Baseline Results:**
| Metric | Value |
|--------|-------|
| Average Walk-Forward Sharpe | 0.18 |
| Win Rate (% positive windows) | 56% |
| Cumulative Return | 5.1% |
| Cointegrated Pairs Found | 48 |
| Crisis Period Sharpe | 1.70 |
| Normal Period Sharpe | -0.26 |

### 2. Improved Strategy (`pairs_trading_improved.ipynb`)

The improved notebook implements five research-backed enhancements:

#### Improvement #1: Pair Persistence Filtering
- Track which pairs remain cointegrated across multiple walk-forward windows
- After window 5, only trade pairs with >60% persistence rate
- Rationale: Persistent pairs have more stable relationships

#### Improvement #2: Financial Sector Focus
- Concentrate on Financial stocks (64% persistence vs 35% average)
- Common interest rate sensitivity creates stable cointegration
- Similar regulatory environment and credit cycle exposure

#### Improvement #3: Dynamic Regime-Based Position Sizing
- Counter-intuitive finding: Crisis periods have HIGHER Sharpe (1.70 vs -0.26)
- VIX > 25 + high persistence pair: 150% position (increase!)
- VIX > 25 + low persistence pair: 25% position (decrease)
- Captures crisis alpha instead of avoiding it

#### Improvement #4: Half-Life Analysis
- Filter pairs by mean reversion speed using AR(1) model
- Only trade pairs with half-life < 20 days
- Faster mean reversion = better execution and more trades

#### Improvement #5: Kelly Criterion Position Sizing
- Optimal capital allocation based on edge
- Formula: `Kelly = (p × b - q) / b`
- Use Half-Kelly (50%) for reduced volatility

**Target Performance:**
| Metric | Baseline | Target |
|--------|----------|--------|
| Sharpe Ratio | 0.18 | 0.40-0.50 |
| Win Rate | 56% | >60% |

## Installation

### Prerequisites
- Python 3.8+
- pip package manager

### Setup

```bash
# Clone the repository
git clone https://github.com/diviprog/pairs_trading_sp.git
cd pairs_trading_sp

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

| Package | Purpose |
|---------|---------|
| yfinance | Download stock data from Yahoo Finance |
| pandas | Data manipulation and analysis |
| numpy | Numerical computing |
| statsmodels | Cointegration tests (Engle-Granger) |
| scipy | Statistical functions |
| matplotlib | Plotting and visualization |
| seaborn | Statistical visualization |
| jupyter | Jupyter notebook environment |
| beautifulsoup4 | Web scraping for S&P 500 list |
| requests | HTTP requests |

## Usage

1. **Start Jupyter Notebook:**
   ```bash
   jupyter notebook
   ```

2. **Run Baseline Analysis:**
   - Open `pairs_trading.ipynb`
   - Run all cells sequentially
   - Results are saved to `data/` folder

3. **Run Improved Strategy:**
   - Open `pairs_trading_improved.ipynb`
   - Requires baseline data files to exist
   - Run all cells sequentially

## Methodology

### Cointegration Testing

The Engle-Granger two-step method tests if two price series share a long-run equilibrium:

1. Regress stock A on stock B to get hedge ratio β
2. Test residuals for stationarity (ADF test)
3. If p-value < 0.05, pair is cointegrated

```python
from statsmodels.tsa.stattools import coint

score, p_value, _ = coint(price_series_A, price_series_B)
```

### Trading Logic

- **Entry**: Z-score crosses ±2.0 standard deviations
- **Exit**: Z-score returns to 0.0
- **Stop Loss**: Z-score exceeds ±4.0

### Walk-Forward Analysis

Prevents look-ahead bias with rolling windows:
- **Training Period**: 24 months (identify cointegrated pairs)
- **Testing Period**: 6 months (out-of-sample trading)
- **Step Size**: 6 months (rolling forward)

## Key Research Findings

1. **In-Sample vs Out-of-Sample Gap**: In-sample Sharpe (2.49) >> Walk-forward Sharpe (0.18)
   - Critical importance of walk-forward testing
   
2. **Crisis Alpha**: Strategy performs BETTER during high VIX periods
   - Crisis Sharpe: 1.70 vs Normal Sharpe: -0.26
   
3. **Persistence Matters**: Only 15% of pairs persist across >60% of windows
   - Persistent pairs are more reliable
   
4. **Sector Effects**: Financial sector pairs show highest persistence (64%)
   - Common factor exposure creates stability

## Limitations

- **Survivorship Bias**: Only current S&P 500 constituents
- **Transaction Costs**: Simplified cost model
- **Execution**: Assumes end-of-day fills
- **Static Hedge Ratio**: Could use Kalman filter for dynamic hedging
- **No Intraday Data**: Daily frequency only

## Future Work

- [ ] Dynamic hedge ratio adjustment (Kalman filter)
- [ ] Intraday execution modeling
- [ ] Alternative universes (Russell 2000, international)
- [ ] GMM-based regime detection (vs fixed VIX thresholds)
- [ ] Machine learning for pair selection

## License

This project is for educational and research purposes.

## Acknowledgments

- Data sourced from Yahoo Finance via `yfinance`
- S&P 500 constituent list from Wikipedia
- Statistical methods from `statsmodels` library
