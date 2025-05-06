# Algo Trading

<details>
<summary>Click to expand/collapse</summary>

## 1. Project Overview

This repository implements and evaluates a trendline‑based breakout trading strategy on minute and hourly Bitcoin price data, enhanced by machine learning and statistical validation.

**Key goals:**

* Clean and enrich tick‑level data
* Fit dynamic support & resistance trendlines
* Generate breakout trades and engineer per‑trade features
* Train a Random Forest to select high‑probability breakouts (meta‑labelling)
* Perform walk‑forward evaluation and true out‑of‑sample testing
* Conduct permutation tests for statistical significance

## 2. Data

1. **Raw minute‑level CSV** (`btcusd_1-min_data.csv`): 2012–present BTC–USD OHLCV at one‑minute intervals.
2. **Hourly CSV** (`BTCUSDT3600.csv`): hourly OHLCV extracts used for early visual exploration.
3. **Train/Test split**:

   * **Train**: 2023‑03‑14 → 2025‑03‑14 (2 years of minute data)
   * **Test**: 2025‑03‑16 → 2025‑05‑04 (\~2 months out‑of‑sample)

## 3. Data Cleaning & Feature Engineering

* **Load** data with `pandas.read_csv`, parse Unix timestamps to `datetime` index.
* **Reindex** to a continuous per‑minute grid; identify and flag missing intervals (\~19 hours gap).
* **Interpolate** small gaps via time‑interpolation on prices; zero‑fill volumes.
* **Feature columns added**:

  * **Log‑prices** (`Open`,`High`,`Low`,`Close` transformed via `np.log`)
  * **Returns**: simple % and log returns
  * **Rolling stats**: moving averages and volatility (std) over multiple windows
  * **Time features**: hour of day, day of week
  * **Technical indicators**: EMA, MACD, RSI

## 4. Trendline Functions

Implemented pure‑Python routines to compute optimal support & resistance lines:

* `check_trend_line(support, pivot, slope, y)` → error if line violates polarity
* `optimize_slope(...)` → numeric search to minimize squared error under polarity constraint
* `fit_trendlines_single(data)` → calls `optimize_slope` for both upper & lower pivots

These yield per‑window slopes and intercepts, used to project trendline levels forward.

## 5. Breakout Strategy

* **`trendline_breakout(close, lookback)`**:

  * Fits support & resistance over the past `lookback` bars
  * Projects levels to current bar
  * Generates a raw signal: +1 above resistance, –1 below support, carry otherwise
* **`trendline_breakout_dataset(ohlcv, lookback, hold_period, tp_mult, sl_mult, atr_lookback)`**:

  * Identifies all breakout trade entries
  * Sets dynamic stops (SL/TP) at ±ATR × multiplier
  * Records per‑trade features:

    * Normalized slope, line error, max distance
    * Normalized volume, ADX value
  * Labels trades by profitability (win=1/lose=0)
  * Outputs `(trades, X, y)` for ML

## 6. Machine Learning & Walk‑Forward

* **Meta‑labeling**: train a Random Forest on `(X_train, y_train)` to predict trade success.
* **Walk‑forward** via `walkforward_model(close, trades, X, y, train_size, step_size)`:

  * Retrain RF every `step_size` bars using all past trades within sliding window
  * Apply to new trade entries, producing `signal` and `prob_signal` time series
* **Single-shot OOS**: train once on 2‑year data, predict on 2‑month test trades in seconds.

## 7. Evaluation & Statistical Testing

* **Metrics**: Profit Factor, Win Rate, cumulative returns vs. buy‑and‑hold.
* **True OOS**: Profit Factor ≈ 1.49, Win Rate ≈ 69% over hold‑out period.
* **Permutation test** (5 000 label shuffles) to compute null distributions of PF & WR.
* **p‑values** ≈ 0.24 (PF) and 0.13 (WR) → insufficient to reject random labeling.

## 8. Conclusions & Next Steps

1. **Insufficient statistical power** with \~2 months OOS; extend test window.
2. **Refine features**: incorporate multi‑timeframe data, volume dynamics, order‑flow proxies.
3. **Alternative validation**: block bootstrap, expanding window CV.
4. **Automated monitoring**: deploy real‑time breakout scanner & retraining schedule.
</details>


# Portfolio Analysis

<details>
<summary>Click to expand/collapse</summary>

## 1. Project Overview

This repository contains a comprehensive portfolio analysis tool that evaluates and visualizes the performance of a diverse set of stocks. The analysis includes key financial metrics, risk assessment, and performance visualization.

**Key Features:**
* Multi-stock data analysis
* Return calculations (simple and logarithmic)
* Risk metrics computation
* Correlation and covariance analysis
* Performance visualization

## 2. Data Sources

* **Stock Data**: Utilizes Yahoo Finance (`yfinance`) for real-time and historical stock data
* **Time Period**: Analyzes stock performance over a specified date range
* **Stocks Analyzed**: Includes major tech companies and other significant market players

## 3. Analysis Components

### 3.1 Data Processing
* Automated data download and cleaning
* Handling of missing values
* Price normalization and standardization

### 3.2 Return Analysis
* Simple returns calculation
* Logarithmic returns computation
* Statistical summary of returns (mean, standard deviation, quartiles)

### 3.3 Risk Assessment
* Volatility calculation
* Correlation matrix generation
* Covariance matrix computation

### 3.4 Visualization
* Price trend analysis (normalized to 100)
* Return distribution plots
* Correlation heatmaps

## 4. Technical Implementation

* **Libraries Used**:
  * `yfinance`: Stock data retrieval
  * `pandas`: Data manipulation and analysis
  * `numpy`: Numerical computations
  * `matplotlib`: Data visualization

* **Key Functions**:
  * Data download and preprocessing
  * Return calculations
  * Statistical analysis
  * Visualization generation

## 5. Future Enhancements

1. **Additional Metrics**:
   * Sharpe ratio
   * Maximum drawdown
   * Value at Risk (VaR)

2. **Portfolio Optimization**:
   * Modern Portfolio Theory implementation
   * Efficient frontier calculation
   * Optimal portfolio weights

3. **Risk Management**:
   * Stop-loss strategies
   * Position sizing
   * Risk-adjusted return metrics

4. **Automation**:
   * Scheduled updates
   * Automated reporting
   * Alert system for significant changes
</details>


# Factor Analysis

<details>
<summary>Click to expand/collapse</summary>

## 1. Project Overview

This repository contains a comprehensive factor analysis framework that implements various factor models to analyze stock returns and risk factors. The analysis includes Fama-French factors, custom factors, and statistical validation of factor exposures.

**Key Features:**
* Multi-factor regression analysis
* Fama-French factor implementation
* Custom factor construction
* Statistical validation and testing
* Portfolio-level and single-stock analysis

## 2. Factor Models

### 2.1 Fama-French Factors
* Market Risk Premium (Mkt-RF)
* Size (SMB - Small Minus Big)
* Value (HML - High Minus Low)
* Risk-free rate adjustments

### 2.2 Custom Factors
* Beta (36-month rolling)
* Volatility (36-month rolling)
* Momentum (11-month cumulative returns)
* Reversal (1-month lagged returns)
* Value (Price-to-Book)
* Growth (Earnings Growth)

## 3. Analysis Components

### 3.1 Data Processing
* Monthly price data aggregation
* Return calculations
* Factor data alignment
* Missing value handling

### 3.2 Statistical Analysis
* OLS regression implementation
* Factor exposure estimation
* Statistical significance testing
* Multicollinearity assessment

### 3.3 Portfolio Analysis
* Multi-stock factor analysis
* Factor exposure decomposition
* Risk attribution
* Performance attribution

## 4. Technical Implementation

* **Libraries Used**:
  * `pandas`: Data manipulation
  * `numpy`: Numerical computations
  * `statsmodels`: Statistical modeling
  * `yfinance`: Market data retrieval
  * `matplotlib` & `seaborn`: Visualization

* **Key Functions**:
  * Factor data processing
  * Regression analysis
  * Statistical testing
  * Results visualization

## 5. Future Enhancements

1. **Additional Factors**:
   * Quality factors
   * Liquidity factors
   * Sentiment factors
   * ESG factors

2. **Methodology Improvements**:
   * Robust regression techniques
   * Time-varying factor exposures
   * Factor timing strategies
   * Machine learning integration

3. **Risk Management**:
   * Factor risk decomposition
   * Factor hedging strategies
   * Risk-adjusted performance metrics
   * Stress testing

4. **Automation**:
   * Automated factor updates
   * Real-time factor monitoring
   * Automated reporting
   * Alert system for significant changes
</details>