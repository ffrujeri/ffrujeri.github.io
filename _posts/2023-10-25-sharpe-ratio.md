---
layout: archive
title: 'Sharpe Ratio'
date: 2023-10-25
permalink: /posts/2023/10/sharpe-ratio
excerpt: "Understanding the Sharpe Ratio"
tags:
  - finance
  - quantitative finance
  - sharpe
---

Most of the quantitative finance measures build on the Markowitz' mean-variance paradigm, which assumes that the mean and standard deviation of the distribution of one-period return are sufficient statistics for evaluating the prospects of an investment portfolio.

The historic Sharpe Ratio is closely realted to the t-statistic for measuring the statistical significance of the mean differential return. The t-statisic will equal the Sharpe Ratio times the square root of T (the number of returns used for the calculations). If historic Sharpe Ratios for a set of funds are computed using the same number of observations, the Sharpe Ratios will thus be proportinal to the t-statistics of the means.

The sharpe Ratio is not independent of the time period over which it is measured. This is true for both ex ante and ex post measures.

$$\sigma_d_T = \sqrt(T) \sigma_d_1$$
$$S_T = \sqrt(T) S_1$$

In practice, the situation is likely to be more complex. Multiperiod returns are usually computed taking compounding into account, which makes the relationship more complicated. Moreover, underlying differential returns may be serially correlated. Even if the underlying process does not involve serial correlation, a specific ex post sample may.

It is common practice to "annualize" data that apply to periods other than one year. Doing so before computing a Sharpe Ratio can provide at least reasonably meaningful comparisons among strategies, even if predictions are intially stated in terms of different measurement periods.

To maximize information content, it is usually desriable to measure risks and returns using fairly short (e.g. montly periods). For purposes of standardization it is then desirable to annualize the results.

For example, an investment in a broad stock market index, financed by borrowing. Typical estimates of the **annual excess return** on the stock market in a developed country might include a mean of 6% per year and a standard deviation of 15%. The resulting excess return **Sharpe ratio** of the "stock market", stated in annual terms would then be 0.40.

It is essential that the Sharpe Ratio is computed using the mean and standard deviation of a **differential return** (or more broadly, the return on what will be termed a zero investment strategy). Otherwise it loses its *raison d'etre*. Clearly, the Sharpe Ration can be considered a special case of the more general construct of the ratio of the mean of any distribution to its standard deviation.

## Correlations
- The ex ante Sharpe Ratio takes into account both the expected differential return and the associated risk, while the ex post version takes into account both the average differential return and the associated variability. Neither incorporates information about the correlation of a fund or strategy with other assets, liabilities, or previous realizations of its own return.

## References
[The Sharpe Ratio](https://web.stanford.edu/~wfsharpe/art/sr/sr.htm)
[The statistics of Sharpe Ratio](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=05561b77acfdd034a585c32048819cc9ba6d1434)

