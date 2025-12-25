# Nonlinear-Portfolio-Optimization-Indonesia-Banks

## Nonlinear Portfolio Optimization with Higher-Order Risk and Transaction Costs for Indonesian Banking Stocks

This repository implements a nonlinear portfolio optimization model for four major Indonesian banking stocks: **BBCA, BBRI, BMRI, and BBNI**, using daily data from Yahoo Finance between **January 2022 and October 2025**.

The model maximizes a custom utility function that jointly accounts for **expected return, variance, skewness, kurtosis**, and **nonlinear transaction and market impact costs** under realistic portfolio constraints.

---

## Project Overview

The project retrieves daily closing prices via `yfinance`, computes daily **log returns**, and derives:

* Annualized mean returns
* Covariance matrix
* Skewness
* Kurtosis

for **BBCA.JK, BBRI.JK, BMRI.JK, and BBNI.JK**.

Using these empirical statistics, a **nonlinear optimization problem** is formulated in **Pyomo** and solved using **IPOPT**, starting from an equal-weight portfolio (25% each). The resulting allocation improves the portfolio utility while respecting all imposed constraints.

---

## Optimization Model

### Decision Variables

Portfolio weights:

$$
w = (w_1, w_2, w_3, w_4)
$$

corresponding to **BBCA, BBRI, BMRI, and BBNI**, with initial weights:

$$
w_{0,i} = 0.25 \quad \forall i
$$

---

### Objective Function

The utility function is defined as:

$$
\begin{aligned}
U(w) =;& \mu^T w

* \lambda_1 w^T \Sigma w

- \lambda_2 \sum_{i=1}^4 \text{skew}*i , w_i^3 \
  &- \lambda_3 \sum*{i=1}^4 \text{kurt}_i , w_i^4

* \sum_{i=1}^4 \left[
  a_i (w_i - w_{0,i})^2

- b_i \sqrt{(w_i - w_{0,i})^2 + \epsilon}
  \right]
  \end{aligned}
  $$

where:

* $\mu$ is the vector of expected returns
* $\Sigma$ is the covariance matrix
* $\text{skew}_i$ and $\text{kurt}_i$ are asset-wise skewness and kurtosis
* $\lambda_1, \lambda_2, \lambda_3$ control variance, skewness, and kurtosis preferences
* $a_i = 10$, $b_i = 0.5$ are transaction cost parameters
* $\epsilon$ is a small constant for numerical stability

---

## Constraints

### Budget Constraint

$$
\sum_{i=1}^4 w_i = 1
$$

---

### Individual Weight Bounds

$$
-0.2 \le w_i \le 0.6 \quad \forall i
$$

This allows limited short-selling and caps exposure to any single stock at 60%.

---

### Leverage Constraint

$$
\sum_{i=1}^4 \sqrt{w_i^2 + \epsilon} \le 1.5
$$

---

### Turnover Constraint

$$
\sum_{i=1}^4 \sqrt{(w_i - w_{0,i})^2 + \epsilon} \le 0.6
$$

This restricts excessive deviation from the initial equal-weight portfolio.

---

### Minimum Sharpe Ratio Constraint

$$
\frac{\mu^T w}{\sqrt{w^T \Sigma w}} \ge 0.25
$$

---

## Implementation in Pyomo

The notebook constructs a **Pyomo `ConcreteModel`** with:

* Continuous decision variables for portfolio weights
* Parameters initialized from empirical estimates
* Nonlinear expressions for risk, higher moments, leverage, and transaction costs

The model is solved using **IPOPT** with multiple random starting points to mitigate local optimum issues. The best solution is selected based on the achieved utility value $U(w)$.

---

## Numerical Experiments and Results

A multi-start procedure is used, recording for each run:

* Optimal portfolio weights
* Utility value
* Expected annual return
* Annual volatility
* Sharpe ratio

The results illustrate how the optimizer balances expected return against variance, skewness, kurtosis, and turnover, with the turnover constraint preventing extreme reallocations away from the initial equal-weight portfolio.

---

## Interpretation

### Risk–Return Trade-Off

The optimized portfolio achieves:

* **Expected annual return:** ≈ 14.4%
* **Annual volatility:** ≈ 23.5%
* **Sharpe ratio:** ≈ 0.61

This exceeds the imposed minimum Sharpe ratio constraint, indicating a strong risk-adjusted performance with a moderately aggressive but controlled risk profile.

---

### Effect of Skewness and Kurtosis

Among the four stocks, **BBNI.JK** exhibits the highest skewness (0.26) and the lowest kurtosis (2.27). Accordingly, it receives the **largest portfolio weight (≈ 0.29)** in the optimal solution.

Assets with **positive skewness** and **lower tail risk** are naturally favored, while **BMRI.JK**, which shows higher kurtosis, receives the smallest allocation. **BBCA.JK** and **BBRI.JK** lie between these extremes both statistically and in portfolio weight.

---

### Impact of Turnover Constraint

The turnover constraint ensures portfolio stability. In the optimal solution, the turnover level is approximately **0.12**, implying only modest deviations from the initial equal-weight portfolio.

This constraint allows the model to respond to differences in return and higher-order risk characteristics while avoiding overly aggressive rebalancing.
