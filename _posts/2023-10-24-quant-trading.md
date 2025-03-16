---
layout: archive
title: 'Quantitative Trading'
date: 2023-10-24
permalink: /posts/2023/10/quant-trading
excerpt: "Quantitative Trading Summary"
tags:
  - finance
  - quantitative finance
  - infrastructure
---

## Performance metrics

The **correct metric to evaluate the performance of a portfolio while accounting for capital exposure** is typically the **risk-adjusted return**. Commonly used metrics in this context include:

---

### 1. **Sharpe Ratio**
   - **Definition**: Measures the return of the portfolio above the risk-free rate, adjusted for the standard deviation (volatility) of the portfolio's returns.
   - **Formula**: 
     \[
     \text{Sharpe Ratio} = \frac{R_p - R_f}{\sigma_p}
     \]
     Where:
     - \(R_p\): Portfolio return
     - \(R_f\): Risk-free rate of return
     - \(\sigma_p\): Standard deviation of portfolio returns
   - **Use**: Evaluates how effectively a portfolio generates returns relative to the risk taken, accounting for capital exposure.

---

### 2. **Sortino Ratio**
   - **Definition**: Similar to the Sharpe Ratio, but focuses only on downside risk (negative deviations), making it a better measure when the concern is avoiding losses.
   - **Formula**: 
     \[
     \text{Sortino Ratio} = \frac{R_p - R_f}{\sigma_d}
     \]
     Where:
     - \(\sigma_d\): Standard deviation of negative returns (downside deviation).
   - **Use**: Highlights the performance of portfolios that avoid large losses while delivering returns.

---

### 3. **Treynor Ratio**
   - **Definition**: Measures the return of a portfolio above the risk-free rate relative to its systematic risk (beta).
   - **Formula**:
     \[
     \text{Treynor Ratio} = \frac{R_p - R_f}{\beta_p}
     \]
     Where:
     - \(\beta_p\): Beta of the portfolio (sensitivity to market movements).
   - **Use**: Suitable for portfolios exposed to systematic (market) risk, particularly when beta is a primary consideration.

---

### 4. **Information Ratio**
   - **Definition**: Measures the portfolio's excess return relative to a benchmark index, adjusted for tracking error.
   - **Formula**:
     \[
     \text{Information Ratio} = \frac{R_p - R_b}{\text{Tracking Error}}
     \]
     Where:
     - \(R_b\): Return of the benchmark index.
   - **Use**: Assesses performance relative to a benchmark while accounting for active risk.

---

### 5. **Capital Exposure Metrics**
   If explicitly considering **capital exposure**:
   - **Profitability Metrics**:
     - **Return on Capital (RoC)**:
       \[
       \text{RoC} = \frac{\text{Net Profit}}{\text{Capital Employed}}
       \]
       Highlights returns relative to the amount of capital exposed in the portfolio.
   - **Drawdown Metrics**:
     - **Maximum Drawdown**: Evaluates the largest peak-to-trough decline in portfolio value, offering insights into capital exposure during downturns.

---

### Choosing the Metric
- **For overall performance with exposure**: Sharpe or Sortino Ratio.
- **For market-sensitive portfolios**: Treynor Ratio.
- **For benchmark comparison**: Information Ratio.
- **For capital-specific evaluation**: RoC or Maximum Drawdown.

Each metric provides unique insights into performance and risk relative to capital exposure, so the best choice depends on the portfolio's strategy and investor objectives.

## References
[Quantitative Trading Summary](https://blog.headlandstech.com/2017/08/)
[Point72](https://github.com/Point72)

