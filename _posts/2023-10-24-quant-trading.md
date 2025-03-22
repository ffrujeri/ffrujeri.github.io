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

### Quantitative Methods
#### Linear regression (1 independent Variable)
  - Standard error of the estimate (smaller SEE indicates a better fit of the model)
$$
SEE = \sqrt{\frac{\sum_{i=1}^{n} (y_i - \hat{y}_i)^2}{n-2}}
$$
  - Prediction interval around the predicted value of the dependent variable
$$
s_f^2 = s^2 \left[1 + \frac{1}{n} + \frac{(x_f - \bar{x})^2}{\sum_{i=1}^{n} (x_i - \bar{x})^2}\right]
$$
$s_f^2$ is the variance of the forecasted value of the dependent variable, $s^2$ is the variance of the residuals, $x_f$ is the value of the independent variable for which the forecast is made, $\bar{x}$ is the mean of the independent variable, and $x_i$ are the values of the independent variable in the sample.

#### Multiple regression (2 or more independent variables)
- Confidence interval for regrssion coefficients: use $n-(k+1)$ degrees of freedom, where $n$ is the number of observations and $k$ is the number of independent variables.

$$
\text{Confidence interval} = \hat{\beta}_j \pm t_{\alpha/2} \times SE(\hat{\beta}_j)
$$
where $t_{\alpha/2}$ is the critical value of the t-distribution with $n-(k+1)$ degrees of freedom at the $\alpha/2$ level of significance, and $SE(\hat{\beta}_j)$ is the standard error of the estimate of the coefficient $\hat{\beta}_j$.

- Hypothesis test on each regression coefficient: use the t-statistic with $n-(k+1)$ degrees of freedom

$$
t_{stat} = \frac{\hat{\beta}_j - \beta_{j,0}}{SE(\hat{\beta}_j)}
$$

- p-value: lowest level of significance at which we can reject the null hypothesis that the population value of the regression coefficient is zero in a two-tailed test (the smaller the p-value, the stronger the evidence against the null hypothesis).
A two-tailed test is used to determine if the coefficient is significantly different from zero in either direction.

- ANOVA table for testing whether all the slope coefficients are simultaneously equal to zero (use a one-tailed F-test with $k$ and $n-(k+1)$ degrees of freedom) and rejecting the null hypothesis if $F_{statistic} > F_{critical} $.

Source of variation | Degrees of freedom | Sum of squares | Mean Sum of Squares | F-statistic | Significance |
--- | --- | --- | --- | --- | --- |
Regression | $k$ | $SSR$ | $MSR = SSR/k$ | $F_{statistic} = MSR/MSE$ | $p < \alpha$ |
Residual | $n-(k+1)$ | $SSE$ | $MSE = SSE/(n-k-1)$ | | |
Total | $n-1$ | $SST$ | | | |

$$
F_{stat} = \frac{MSR}{MSE} = \frac{SSR/k}{SSE/(n-(k+1))}
$$

- Standard error of the estimate: $$SSER = \sqrt{MSE}$$ using MSE from the ANOVA table.
- Coefficient of determination: $$R^2 = \frac{\text{Total variation - Unexplained variation}}{\text{Total variation}} = 1 - \frac{SSE}{SST}$$
where $SST$ is the total sum of squares

- Adjusted $R^2$: $$R^2_{adj} = 1 - \frac{SSE/(n-k-1)}{SST/(n-1)} = 1 - \frac{(1-R^2)(n-1)}{n-k-1}$$
the adjusted $R^2$ is used to compare models with different numbers of independent variables. It penalizes the addition of independent variables that do not improve the model.

#### Violations of the assumptions of the linear regression model
1. **Linearity**: The relationship between the independent and dependent variables is linear.
2. **Independence**: The residuals are independent of each other.
3. **Homoscedasticity**: The residuals have constant variance.
    - Unconditional heteroscedasticity: the variance of the residuals is not constant.
    - Conditional heteroscedasticity: the variance of the residuals is not constant after conditioning on the independent variables.
4. **Normality**: The residuals are normally distributed.
5. **No multicollinearity**: The independent variables are not highly correlated with each other.
6. **No autocorrelation**: The residuals are not correlated with each other.
7. **Model specification**: The model is correctly specified.
    - Misspecified functional form: the relationship between the independent and dependent variables is not correctly specified.
      - Omitted variables: important independent variables are not included in the model.
 - Time-series misspecification: 
    - Including lagged dependent variables or autocorrelation terms can help address this issue.
    - Including an independent variable that is a function of the dependent variable
    - Measuring independent variables with error
    - Nonstationarity: the mean, variance, or autocorrelation of the residuals changes over time.

    - The Durbin-Watson statistic tests for autocorrelation in the residuals.
    - The Breusch-Godfrey test is used to test for autocorrelation in the residuals in a multiple regression model.
    - The ARCH test is used to test for conditional heteroscedasticity in the residuals.
    - The Jarque-Bera test is used to test for normality in the residuals.
    - The Ramsey RESET test is used to test for misspecified functional form in the model.


#### Time series analysis
- Linear tred model: $$Y_t = \beta_0 + \beta_1 t + \epsilon_t$$
The dependent variable grows by a constant amount each period.
- Log-linear model: $$\ln(Y_t) = \beta_0 + \beta_1 t + \epsilon_t$$
- Quadratic trend model: $$Y_t = \beta_0 + \beta_1 t + \beta_2 t^2 + \epsilon_t$$
The dependent variable grows by an increasing or decreasing amount each period.
- Seasonal model: $$Y_t = \beta_0 + \beta_1 t + \sum_{i=1}^{s-1} \beta_i D_{it} + \epsilon_t$$
where $D_{it}$ is a dummy variable that takes the value of 1 in the $i$th season and 0 otherwise.

- Autoregressive model: uses past values of the dependent variable to predict its current value
  - First-order autoregressive model:
$$Y_t = \beta_0 + \beta_1 Y_{t-1} + \epsilon_t$$
  - Second-order autoregressive model:
$$Y_t = \beta_0 + \beta_1 Y_{t-1} + \beta_2 Y_{t-2} + \epsilon_t$$

- AR model must be covariance stationary: the mean, variance, and autocorrelation of the residuals are constant over time. And specified such that the error terms do not exhibit serial correlation and heteroscedasticity in order to be used for statistical inference.

- t-test for serial (auto) correlation: $$t_{stat} = \frac{\hat{\rho}_1}{SE(\hat{\rho}_1)}$$
where $\hat{\rho}_1$ is the estimated first-order autocorrelation coefficient and $SE(\hat{\rho}_1)$ is the standard error of the estimate of the first-order autocorrelation coefficient.

- Mean-reverting level of AR(1) model
$$
\text{Mean-reverting level} = \frac{\beta_0}{1-\beta_1}
$$

- Random walk is a special case of the AR(1) model where $\beta_1 = 1$ and $\beta_0 = 0$ that is not covariance stationary (undefinied mean reverting level).
$$
Y_t = Y_{t-1} + \epsilon_t
$$
where $\epsilon_t$ is a white noise error term. $E(\epsilon_t) = 0$ and $Var(\epsilon_t) = \sigma^2$, and the error terms are uncorrelated with each other $E(\epsilon_t \epsilon_{t-1}) = 0$.

- First difference of a random walk is stationary
$$
\Delta Y_t = Y_t - Y_{t-1} = \epsilon_t
$$

- AR(1) model has a unit root if the slope coefficient is equal to 1. The unit root test is used to determine if the time series is stationary or nonstationary.

- ARCH models: used to determine whether the variance of the residuals is constant over time.
  - ARCH(1) model:
$$
\epsilon_t = \sigma_t Z_t
$$
- if ARCH errors are found, use generalized least squares to correct for heteroscedasticity. A GARCH model can be used to model the conditional variance of the residuals.
  - GARCH(1,1) model:
$$
\sigma_t^2 = \alpha_0 + \alpha_1 \epsilon_{t-1}^2 + \beta_1 \sigma_{t-1}^2
$$
where $\alpha_0$ is the constant term, $\alpha_1$ is the coefficient of the squared residuals, $\beta_1$ is the coefficient of the previous period's variance, and $\sigma_{t-1}^2$ is the previous period's variance.

- Dickey-Fuller test for unit root
  - Null hypothesis: the time series has a unit root and is nonstationary.
  - Alternative hypothesis: the time series is stationary.
  - Test statistic: the Dickey-Fuller test statistic is compared to critical values to determine if the null hypothesis can be rejected.
  - Critical values: the critical values depend on the number of observations and the significance level of the test.
  - Conclusion: if the test statistic is less than the critical value, the null hypothesis is rejected, and the time series is stationary.

#### Machine Learning
- Penalized regression: Lasso (L1 regularization) and Ridge (L2 regularization) regression are used to prevent overfitting by penalizing the size of the coefficients.

- Support vector machines (SVM): used for classification and regression tasks by finding the hyperplane that maximizes the margin between classes.

- K-nearest neighbors (KNN): used for classification and regression tasks by finding the K-nearest neighbors to a data point and using their values to make predictions.

- Decision trees: used for classification and regression tasks by recursively splitting the data into subsets based on the values of the independent variables.

- Random forests: an ensemble method that uses multiple decision trees to make predictions by averaging the results of individual trees.


## References
[Quantitative Trading Summary](https://blog.headlandstech.com/2017/08/)
[Point72](https://github.com/Point72)

