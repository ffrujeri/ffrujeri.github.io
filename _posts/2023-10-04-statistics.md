---
layout: archive
title: 'Basic Statistics Course'
date: 2023-10-04
permalink: /posts/2023/10/statistics
excerpt: "Basic Statistics Course"
tags:
  - statistics
---

## Descriptive Statistics
- mean
- median / percentiles
- std-dev
- skewness
- kurtosis

## Simple linear regression model
We are now ready to summarize the four conditions that comprise "the simple linear regression model:"

**L**inear Function: The mean of the response, $\mbox{E}(Y_i)$ at each value of the predictor $x_i$ is a Linear function of the $x_i$.

**I**ndependent: The errors, $\epsilon_{i}$, are Independent. 
**N**ormally Distributed: The errors, $\epsilon_{i}$, at each value of the predictor, $x_i$, are Normally distributed.
**E**qual variances (denoted $\sigma^{2}$): The errors, $\epsilon_{i}$, at each value of the predictor, $x_i$, have Equal variances.

An equivalent way to think of the first (linearity) condition is that the mean of the error, $\mbox{E}(\epsilon_i)$, at each value of the predictor, $x_i$, is *zero*. An alternative way to describe all four assumptions is that the errors, $\epsilon_{i}$, are independent normal random variables with mean zero and constant variance, $\sigma^{2}$.

## Chi-squared tests

## Maxium Likelihood estimation
Maximum Likelihood Estimation (MLE) is a method used in statistics to estimate the parameters of a statistical model. The idea behind MLE is to find the parameter values that maximize the likelihood function, which measures how well the model explains the observed data.

### Likelihood Function
The likelihood function, denoted as $L(\theta;x)$, represents the probability of observing the given data $𝑥$, given a set of parameters $𝜃$ for a statistical model. Mathematically, the likelihood function is defined as:

$$L(\theta;x) = f(x|\theta)$$
where $f(x|\theta)$ is the pdf or pmf of the data given the parameters.

$$\hat{\theta} = \arg\max_{\theta} L(\theta; x)$$

- Example:
Let's consider a simple example with a normal distribution. Suppose we have a sample fo $n$ points from a normal distribution with unknown mean $\mu$ and a known variance $\sigma^2$ 

$$L(\mu; x) = \prod_{i=1}^{n} \frac{1}{\sqrt{2\pi\sigma^2}} \exp\left(-\frac{(x_i - \mu)^2}{2\sigma^2}\right)$$

## Degrees of Freedom
In general, degrees of freedom refer to the number of independent pieces of information that are available to estimate a parameter or calculate a statistic. The concept is often used to describe the number of values in the final calculation of a statistic that are free to vary without violating any constraints.

### Properties
- Consistency: Under certain regularity conditions, the MLE estimators are consistent, which means that as the sample size increases, the estimates converge to the true parameter values.

- Asymptotic Normality: For large samples, the MLE estimators are approximately normally distributed, which allows for the construction of confidence intervals and hypothesis tests.

- Efficiency: Among all unbiased estimators, the MLE provides the minimum variance, making it efficient.

## References
[Penn State Eberly College of Science](https://online.stat.psu.edu/stat501/lesson/1/1.3)

[zedstatistics](https://www.youtube.com/watch?v=N20rl2llHno)