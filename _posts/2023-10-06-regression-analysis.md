---
layout: archive
title: 'Regression Analysis'
date: 2023-10-06
permalink: /posts/2023/10/regression-analysis
tags:
  - statistics
  - regression
---

## Linear Regression

In linear regression analysis, the correlation and the beta coefficient (also known as the regression coefficient) are essential concepts that help us understand the relationship between the independent variable X and the dependent variable 𝑌

## Correlation

**Definition:** Correlation measures the strength and direction of a linear relationship between two variables `X` and `Y`. It is represented by the correlation coefficient, usually denoted as `r`.

**Range:** The value of `r` ranges from `-1` to `1`:

- `r=1`: Perfect positive correlation (as `X` increases, `Y` also increases).
- `r=-1`: Perfect negative correlation (as `X` increases, `Y` decreases).
- `r=0`: No correlation (no linear relationship).

**Calculation:** The correlation coefficient can be calculated using the formula:

$$r = \frac{\text{Cov}(X,Y)}{\sigma_X \cdot \sigma_Y} = \frac{n\sum{XY} - (\sum{X})(\sum{Y})}{\sqrt{[n\sum{X^2} - (\sum{X})^2] \cdot [n\sum{Y^2} - (\sum{Y})^2]}}$$

## Beta Coefficient

**Definition:** The beta coefficient (β) represents the change in the dependent variable `Y` for a one-unit change in the independent variable `X`. It quantifies the   
 relationship between `X` and `Y` in a linear regression model.

**Interpretation:**

- If β > 0: There is a positive relationship; as `X` increases, `Y` tends to increase.
- If β < 0: There is a negative relationship; as `X` increases, `Y` tends to decrease.
- If β = 0: No relationship between `X` and `Y`.

**Calculation:** The beta coefficient can be calculated using the formula:

$$
β = \frac{\sigma_X}{\sigma_Y} \cdot r = \frac{\sum{(X - \bar{X})(Y - \bar{Y})}}{\sum{(X - \bar{X})^2}}
$$

where:

- `β`: the beta coefficient
- `X`: the independent variable
- `Y`: the dependent variable
- `\bar{X}`: the mean of `X`
- `\bar{Y}`: the mean of `Y`

### Relationship Between Correlation and Beta Coefficient

The correlation coefficient and the beta coefficient are related in that the beta coefficient can be derived from the correlation coefficient when considering the standard deviations of `X` and `Y`. In other words, the beta coefficient captures how much `Y` changes for a unit change in `X`, while the correlation coefficient measures the strength of their linear relationship.

## Scaled Beta (sb)

**Definition:**

The scaled beta (sb) is a standardized measure that quantifies the relationship between an independent variable (X) and a dependent variable (Y) in a linear regression model. It is calculated as:

$$sb = r \cdot \sigma_Y$$


where:

- `r` is the correlation coefficient between X and Y.
- `σ_Y` is the standard deviation of Y.

**Interpretation:**

- **Magnitude:** The magnitude of sb indicates the strength of the relationship between X and Y, scaled by the variability of Y. A larger absolute value of sb implies a stronger relationship.
- **Sign:** The sign of sb is the same as the sign of r:
  - If sb > 0: There is a positive relationship (as X increases, Y tends to increase).
  - If sb < 0: There is a negative relationship (as X increases, Y tends to decrease).
  - If sb = 0: There is no relationship between X and Y.

**Comparison with Beta Coefficient:**

The beta coefficient (β) is a measure of the change in Y for a one-unit change in X, while the scaled beta (sb) is a measure of the strength of the relationship between X and Y, scaled by the variability of Y. The relationship between β and sb is:

$$β = r \cdot \sigma_Y / σ_X$$

where `σ_X` is the standard deviation of X.

**Importance of Scaled Beta:**

- **Ease of Interpretation:** sb is easier to interpret in terms of how much change in Y is expected with a change in X, especially when focusing on the variability of Y.
- **Standardization:** It allows for comparison across different datasets or variables, even if they have different scales or units.
- **Direct Relation to Correlation:** sb directly depends on the correlation coefficient, highlighting the importance of both correlation and variability in understanding the relationship between X and Y.

- The scaled beta (sb) provides a valuable measure of the relationship between the independent variable (X) and the dependent variable (Y), focusing on the strength and direction of this relationship in terms of the variability of Y. It connects the concepts of correlation and regression, emphasizing how variability in Y influences the predictive power of X.

## Example Scenario

Suppose we have a dataset that captures the relationship between the number of hours studied (independent variable X) and scores obtained in an exam (dependent variable Y). Here's a sample dataset:

| Hours Studied (X) | Exam Score (Y) |
|---|---|
| 1 | 50 |
| 2 | 60 |
| 3 | 70 |
| 4 | 80 |
| 5 | 90 |

**Step 1: Calculate Correlation**

To calculate the correlation coefficient r, we can use the formula mentioned earlier:

$$r = \frac{n(\sum XY) - (\sum X)(\sum Y)}{\sqrt{[n\sum X^2 - (\sum X)^2][n\sum Y^2 - (\sum Y)^2]}}$$


**Calculations:**

* n = 5 (number of data points)
* ∑X = 15
* ∑Y = 350
* ∑XY = 1150
* ∑X^2 = 55
* ∑Y^2 = 25500

Now substitute into the correlation formula:
$$r = \frac{5(1150) - (15)(350)}{\sqrt{[5 \cdot 55 - (15)^2][5 \cdot 25500 - (350)^2]}}$$

$$ r = \frac{500}{\sqrt{50 \cdot 5000}} = \frac{500}{\sqrt{250000}} = \frac{500}{500} = 1$$

$$ β = \frac{r \cdot \sigma_Y}{\sigma_X} $$

**Standard Deviations**

Calculate σ_Y (Standard deviation of Y):

$$σ_Y = \sqrt{\frac{\sum (Y_i - \bar{Y})^2}{n-1}}$$

Where $\bar{Y} = \frac{\sum Y}{n} = \frac{350}{5} = 70$

Now calculating:
$$σ_Y = \sqrt{\frac{(50 - 70)^2 + (60 - 70)^2 + (70 - 70)^2 + (80 - 70)^2 + (90 - 70)^2}{4}} = \sqrt{250} \approx 15.81$$

Calculate σ_X (Standard deviation of X):
$$ σ_X = \sqrt{\frac{(1 - 3)^2 + (2 - 3)^2 + (3 - 3)^2 + (4 - 3)^2 + (5 - 3)^2}{4}} = \sqrt{2.5} \approx 1.58$$

Now Calculate β:
$$ β = \frac{1 \cdot 15.81}{1.58} \approx 10 $$

**Step 3: Calculate Scaled Beta**

Finally, we can calculate the scaled beta sb:
$$ sb = r \cdot \sigma_Y = 1 \cdot 15.81 = 15.81 $$

**Summary of Results**

* Correlation (r): 1
* Beta Coefficient (β): 10
* Scaled Beta (sb): 15.81

**Interpretation of Results**

* **Correlation (r = 1):** This indicates a perfect positive linear relationship between the number of hours studied and exam scores. As hours increase, scores increase without exception.
* **Beta Coefficient (β = 10):** This means that for each additional hour studied, the exam score increases by an average of 10 points.
* **Scaled Beta (sb = 15.81):** This reflects the strength of the relationship, indicating that the increase in exam scores, when scaled by the standard deviation of scores, is significant.

## Scaled Beta vs. Beta Coefficient: A Comparative Analysis

**Dataset 1:**

* Correlation (r): 0.9
* Scaled Beta (sb): 14.23

**Dataset 2:**

* Correlation (r): 0.9
* Scaled Beta (sb): 1.42

**Interpretation**

**Correlation (r = 0.9):**

In both datasets, the correlation coefficient indicates a strong linear relationship between hours studied (X) and exam scores (Y). This means that as the number of hours studied increases, the exam scores tend to increase as well. However, this does not capture the variability of Y.

**Scaled Beta (sb):**

* **Dataset 1:** The scaled beta of 14.23 suggests that even a small change in hours studied is associated with a significant change in exam scores, considering the high variability in scores. It indicates that the relationship is strong relative to the spread of the scores.
* **Dataset 2:** The scaled beta of 1.42 indicates a much weaker relationship in the context of the variability in scores. Here, while there is still a strong correlation, the actual change in exam scores for an increase in hours studied is much less significant due to the lower variability of scores.
Use code

This example illustrates how scaled beta (sb) contextualizes the relationship between X and Y by accounting for the spread of Y.

* **High sb:** Indicates that changes in X have substantial implications for Y given the spread of Y, emphasizing the strength of the relationship.
* **Low sb:** Suggests that while X and Y may still be strongly correlated, the actual impact of changes in X on Y is less significant due to the low variability in Y.

Thus, scaled beta provides a more nuanced understanding of the relationship, helping to interpret how meaningful the correlation is in practical terms.

**Correlation** provides a measure of the strength and direction of the linear relationship between two variables, but it does not account for the variability of the dependent variable (Y). This can lead to misinterpretations in practical applications.

**Scaled Beta** contextualizes the correlation by providing insight into the expected change in Y in relation to its typical variability. This allows for a more comprehensive understanding of how changes in X affect Y in practical terms, particularly when Y varies significantly.

**In summary:**

* **Correlation** measures the strength of the linear relationship.
* **Scaled Beta** provides a standardized measure that considers both the strength of the relationship and the variability of Y.

By using both correlation and scaled beta, you can gain a more complete understanding of the relationship between two variables and make more informed interpretations in practical applications.
Use code with caution.











## References
