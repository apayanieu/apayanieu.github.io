---
title: "Monte Carlo Simulation for Option Pricing"
excerpt: "A Monte Carlo study of European and Asian option pricing under Black–Scholes, with a geometric-Asian control variate and calibration to real SPY option prices."
collection: portfolio
tags:
  - Monte Carlo
  - Quantitative Finance
  - Options
  - Variance Reduction
  - Python
use_math: true
---

A numerical-methods project applying Monte Carlo simulation to option pricing under the Black–Scholes framework, with a focus on both European and arithmetic Asian call options. The analysis studies convergence of the estimators as the number of paths and time discretization are refined, incorporates variance reduction through a geometric Asian control variate, and calibrates the model to real SPY option data via historical volatility estimation and implied volatility computation.

## Overview

This project builds a single, self-contained workflow for pricing financial derivatives numerically. Three questions drive the study:

1. **European call pricing** — How close does a plain Monte Carlo estimator get to the closed-form Black–Scholes price, and how does the error scale with the number of simulated paths?
2. **Asian call pricing** — Path-dependent options have no simple closed form; what does Monte Carlo produce, and how sensitive is the price to the time discretization?
3. **Variance reduction and real-market calibration** — Can a highly correlated control variate (the geometric Asian call, which *does* have a closed form) sharpen the estimator without extra simulation, and how do the model prices compare against real SPY option quotes?

## Setup

All simulations use the following Black–Scholes parameters:

| Parameter | Value | Meaning |
|---|---|---|
| $S_0$ | 100 | Initial stock price |
| $K$ | 100 | Strike (at-the-money) |
| $r$ | 0.05 | Risk-free rate (annual) |
| $\sigma$ | 0.20 | Volatility (annual, constant) |
| $T$ | 1.0 | Maturity (years) |
| $M$ | 252 | Time steps for path-based simulations |

Sample sizes $N$ span $10^3$ to $10^6$ paths, with a fixed RNG seed for reproducibility.

## Methodology

### Geometric Brownian Motion paths

Under the risk-neutral measure, the stock follows

$$
S_{t_{k+1}} = S_{t_k} \exp\left[\left(r - \tfrac{1}{2}\sigma^2\right)\Delta t + \sigma \sqrt{\Delta t}\, Z_k\right],
\qquad Z_k \sim \mathcal{N}(0,1), \quad \Delta t = T/M.
$$

Paths are simulated by cumulatively summing log-increments in vectorized NumPy, which handles $10^6$ paths across 252 steps in a single array operation.

### European call: Monte Carlo vs Black–Scholes

The plain Monte Carlo estimator for a European call is

$$
\hat{C}_{\text{MC}} = e^{-rT} \cdot \frac{1}{N}\sum_{i=1}^{N} \left(S_T^{(i)} - K\right)^{+},
\qquad
\widehat{\text{SE}} = \frac{s_X}{\sqrt{N}},
$$

where $s_X$ is the sample standard deviation of the discounted payoffs. The analytic benchmark is the Black–Scholes formula

$$
C_{BS} = S_0 \,\Phi(d_1) - K e^{-rT} \Phi(d_2),
\quad
d_1 = \frac{\ln(S_0/K) + (r + \sigma^2/2)T}{\sigma\sqrt{T}},
\quad
d_2 = d_1 - \sigma\sqrt{T}.
$$

### Asian call: arithmetic average

The payoff of an arithmetic-average Asian call depends on the *path*:

$$
A^{(i)} = \frac{1}{M}\sum_{k=1}^{M} S_{t_k}^{(i)},
\qquad
\hat{C}_{\text{Asian}} = e^{-rT} \cdot \frac{1}{N}\sum_{i=1}^{N} \left(A^{(i)} - K\right)^{+}.
$$

Because the arithmetic average of log-normals is not log-normal, no simple Black–Scholes-style closed form exists. This is the natural test bed for Monte Carlo.

### Control variate: geometric Asian

The *geometric* Asian call — with payoff $\left(\prod_{k=1}^{M} S_{t_k}\right)^{1/M}$ — **does** have a closed form (Kemna–Vorst):

$$
\sigma_n^2 = \sigma^2 \frac{(n+1)(2n+1)}{6n^2},
\qquad
\mu_n = \left(r - \tfrac{1}{2}\sigma^2\right)\frac{n+1}{2n} + \tfrac{1}{2}\sigma_n^2,
$$

$$
C_{\text{geo}} = e^{-rT}\left[ S_0 e^{\mu_n T} \Phi(d_1) - K \Phi(d_2) \right],
\quad d_1 = \frac{\ln(S_0/K) + (\mu_n + \tfrac{1}{2}\sigma_n^2)T}{\sigma_n \sqrt{T}},
\quad d_2 = d_1 - \sigma_n\sqrt{T}.
$$

Because arithmetic and geometric averages of the same path are almost perfectly correlated ($\rho \approx 0.9996$ in practice), the geometric payoff is a near-ideal control variate. The adjusted estimator is

$$
X_i^{\text{CV}} = X_i + \hat{\beta}\left(C_{\text{geo}} - Y_i\right),
\qquad
\hat{\beta} = \frac{\widehat{\text{Cov}}(X, Y)}{\widehat{\text{Var}}(Y)},
$$

where $X_i$ is the arithmetic payoff and $Y_i$ the geometric payoff on the same path.

### Calibration to SPY data

To connect the model to real markets, historical SPY returns are used to estimate realized volatility, which feeds into the pricing formulas. Model prices are then compared against observed SPY option market prices, and implied volatility is extracted by inverting the Black–Scholes formula. This closes the loop between the simulation framework and market data.

## Results

### European call — convergence in $N$

The Black–Scholes benchmark is

$$
C_{BS} = 10.450584.
$$

Monte Carlo estimates for increasing $N$:

| $N$ | $\hat{C}$ | SE | Relative error |
|---|---|---|---|
| 1,000 | 9.911980 | 0.448345 | −5.15% |
| 5,000 | 10.168920 | 0.205861 | −2.70% |
| 20,000 | 10.544517 | 0.105179 | +0.90% |
| 100,000 | 10.420541 | 0.046770 | −0.29% |
| 500,000 | 10.450050 | 0.020856 | −0.005% |
| 1,000,000 | 10.453195 | 0.014731 | +0.025% |

Two features stand out. First, the **relative error shrinks roughly as $N^{-1/2}$**, exactly as the standard Monte Carlo error scaling predicts — the log–log plot of absolute error versus $N$ is close to a straight line of slope $-1/2$. Second, at $N = 10^6$ the estimator is within $0.03\%$ of the analytic price, and the analytic price lies well inside the $\pm 1.96 \cdot \text{SE}$ band.

![MC estimate of the European call price vs N, with 95% CI and the Black–Scholes benchmark](/images/mc-european-convergence.png)

![Absolute error vs N on log–log axes, with an $N^{-1/2}$ reference line](/images/mc-european-error.png)

### Asian call — pricing and discretization

The arithmetic Asian call price, for $N = 2 \times 10^5$ paths and varying numbers of time steps $M$:

| $M$ | Price | SE | Error vs ref |
|---|---|---|---|
| 16 | 6.046313 | 0.018659 | 0.275 |
| 32 | 5.876889 | 0.018224 | 0.106 |
| 64 | 5.848025 | 0.018052 | 0.077 |
| 126 | 5.813672 | 0.017960 | 0.043 |
| 252 | 5.758028 | 0.017828 | 0.013 |
| 504 | 5.750807 | 0.017852 | 0.020 |
| 1008 | 5.771160 | 0.017908 | 0.000 (ref) |

Two things to note:

- The **Asian price is well below the European call**. Averaging along the path dampens the right tail of $S_T$, so the option is worth less — the standard "Asian smoothing" effect.
- The **discretization bias falls quickly with $M$** and is already within the Monte Carlo noise by $M \approx 252$. The remaining scatter across $M \geq 252$ is consistent with estimator variance, not systematic bias.

A higher-precision reference (M = 252, N = 10⁶) gives

$$
\hat{C}_{\text{Asian}} = 5.772948 \pm 0.008005.
$$

![Arithmetic Asian call price vs number of time steps M](/images/mc-asian-M-convergence.png)

### Control variate — variance reduction

Using the same 200,000 paths, the geometric Asian has a closed-form price of $5.565509$, and the simulated geometric payoff reproduces it to within sampling error. Regressing the arithmetic payoff on the geometric payoff gives an almost perfect fit.

| Estimator | Price | SE |
|---|---|---|
| Plain Monte Carlo (arithmetic) | 5.786390 | 0.017933 |
| Control variate (geometric) | 5.781998 | 0.000494 |

- **Correlation** $\rho(X, Y) \approx 0.9996$
- **Variance reduction factor** $\approx 1{,}318\times$

In other words, the control variate delivers the same precision as a plain Monte Carlo run roughly **1,300 times larger**. The point estimates differ by about $0.004$, well inside the plain MC standard error of $0.018$ — confirming that the control variate does not bias the estimator, only reduces its variance.

![Distribution of terminal prices S_T vs path averages S̄ (the averaging effect)](/images/mc-ST-vs-Savg.png)

## Discussion

The results tell a coherent story about Monte Carlo as a pricing engine:

1. **Bias-free but noisy.** The plain European estimator has essentially no discretization bias (terminal $S_T$ is exact), and the residual error is pure sampling noise scaling as $N^{-1/2}$. Doubling precision requires quadrupling paths — the well-known Monte Carlo tax.
2. **Path dependence is where Monte Carlo earns its keep.** The Asian option has no closed form, but the simulation framework handles it without modification. Discretizing the time axis introduces a small bias that is dominated by MC noise once $M \gtrsim 252$.
3. **Control variates are the highest-leverage tool in the toolkit.** A good control variate does not just shave a bit off the standard error — it can change the *effective sample size* by orders of magnitude. Here, the geometric Asian is almost a perfect proxy for the arithmetic payoff because both are functionals of the same underlying path.

For a production pricing desk, this suggests a natural rule of thumb: when a path-dependent payoff has a related payoff with a closed form, use the closed-form payoff as a control variate *before* reaching for more paths.

## Limitations

- **Constant volatility.** The Black–Scholes model is the toy version of reality. Real markets exhibit volatility smiles, stochastic volatility, and jumps — none of which are captured here. Adding these would change both the pricing and the convergence behaviour.
- **No early exercise or American features.** The framework prices only European- and Asian-style payoffs. American and Bermudan options require a different methodology (LSMC, PDE, or trees).
- **Monte Carlo error scales as $N^{-1/2}$.** This is fundamental. For high-dimensional problems where Monte Carlo is the only viable tool, more sophisticated variance reduction — antithetic variates, stratified sampling, quasi-Monte Carlo — would be the next lever to pull.
- **Historical-data scope.** Realized volatility is estimated from a fixed window of SPY returns and treated as constant through maturity; the model does not incorporate time-varying volatility or regime shifts.
- **Discretization of the average.** The arithmetic average is taken over the fixed grid $t_1, \ldots, t_M$ (discretely monitored), not as a continuous integral. Real Asian options are specified in discrete fixing terms, so this is realistic — but it does mean the price is a function of the fixing schedule, not just of the underlying.

---

**Language:** Python  
**GitHub repo:** [apayanieu/montecarlo_nm](https://github.com/apayanieu/montecarlo_nm)
