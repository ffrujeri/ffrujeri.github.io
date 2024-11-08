---
layout: archive
title: 'Regularization in Machine Learning'
date: 2023-10-06
permalink: /posts/2023/10/regression-analysis
excerpt: "Discussion about Lasso, Ridge, and Elastic Net regularization techniques in machine learning."
tags:
  - regularization
  - l1-regularization
  - l2-regularization
  - elastic-net
---

## Elastic-Net
### ElasticNet Objective Function (Updated Notation):

$$ \min_{\beta} \left( \frac{1}{2} | y - X \beta |_2^2 + \lambda \alpha | \beta |_1 + \frac{\lambda (1 - \alpha)}{2} | \beta |_2^2 \right) $$

where:
- $\lambda$ is the overall regularization strength,
- $\alpha \in [0, 1]$ is the mixing parameter:
- When $\alpha = 1$, ElasticNet reduces to Lasso (only L1 penalty),
- When $\alpha = 0$, it reduces to Ridge regression (only L2 penalty).

### Update Rule for ElasticNet:
We focus on each coefficient $\beta_j$ in a coordinate descent approach, keeping the other coefficients fixed. Let’s derive the update for $\beta_j$.
1. Residual Sum of Squares (RSS):

    - The derivative with respect to $\beta_j$ is: $$X_j^T (y - X \beta) + | X_j |_2^2 \beta_j $$

    - The residual $r$ is defined as: $$ r = y - X \beta $$ which can be expanded as: $$ y - X \beta = y - \sum_{k=1}^p X_k \beta_k $$ 

where $X_k$ is the $k$-th column of $X$, and $\beta_k$ is the corresponding coefficient.
    
- To isolate the contribution of $\beta_j$, we split the residual into two parts:
  - The part involving $\beta_j$,
  - The part involving all other coefficients $\beta_{-j}$ (where $\beta_{-j}$ means all coefficients except $\beta_j$).
Thus, we can rewrite the residual as: 
$$ y - X \beta = y - X_j \beta_j - \sum_{k \neq j} X_k \beta_k = (y - X_{-j} \beta_{-j}) - X_j \beta_j $$ 

where $X_{-j} \beta_{-j}$ is the contribution of all predictors except $X_j$.

- Rewriting the Partial Derivative for Coordinate Descent:

In coordinate descent, we aim to update $\beta_j$ while keeping all other coefficients fixed. The gradient of the residual sum of squares term with respect to $\beta_j$ is:

$$ X_j^T (y - X \beta) $$
By substituting $y - X \beta$ with $(y - X_{-j} \beta_{-j}) - X_j \beta_j$, we get:

$$ X_j^T \left( (y - X_{-j} \beta_{-j}) - X_j \beta_j \right) $$

Expanding this term gives:

$$ X_j^T (y - X_{-j} \beta_{-j}) + X_j^T X_j \beta_j $$

Rearranging:
Now, we isolate the term involving \beta_j:

$$ X_j^T (y - X_{-j} \beta_{-j}) - | X_j |_2^2 \beta_j $$

This term, $X_j^T (y - X_{-j} \beta_{-j})$, is often called the partial residual for $\beta_j$, as it represents the residual $y - X \beta$ without the contribution of $X_j \beta_j$.


2. L1 Penalty:

    - The derivative of $\lambda \alpha | \beta |_1$ with respect to $\beta_j$ is $\lambda \alpha \cdot \text{sign}(\beta_j)$

3. L2 Penalty:

    - The derivative of $\frac{\lambda (1 - \alpha)}{2} | \beta |_2^2$ with respect to $\beta_j$ is $\lambda (1 - \alpha) \beta_j$.

4. Combining these, the partial derivative of the objective function with respect to $\beta_j$ is:

$$ \frac{\partial \mathcal{L}}{\partial \beta_j} = - X_j^T (y - X \beta) + | X_j |_2^2 \beta_j + \lambda (1 - \alpha) \beta_j + \lambda \alpha \cdot \text{sign}(\beta_j) $$

5. To update $\beta_j$, we set the partial derivative to zero and rearrange:

$$ \beta_j \left( | X_j |2^2 + \lambda (1 - \alpha) \right) = X_j^T (y - X{-j} \beta_{-j}) - \lambda \alpha \cdot \text{sign}(\beta_j) $$

where $X_{-j} \beta_{-j}$ represents the prediction excluding $X_j \beta_j$.

Dividing through by $| X_j |_2^2 + \lambda (1 - \alpha)$, we get:

$$ \beta_j = \frac{S \left( X_j^T (y - X_{-j} \beta_{-j}), \lambda \alpha \right)}{| X_j |_2^2 + \lambda (1 - \alpha)} $$

where $S(z, \lambda)$ is the soft-thresholding operator defined as:
$$ S(z, \lambda) = \text{sign}(z) \cdot \max(|z| - \lambda, 0) $$



## References
