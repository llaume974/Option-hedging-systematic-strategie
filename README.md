# Kernel Distribution, Option Covering & Systematic Strategy

**Authors:** Guillaume Attila
**Supervisor:** Matthieu Garcin  
**Date:** March 16, 2024

> This README is a Markdown adaptation of the report “Kernel distribution, option covering and systematic strategy.”

---

## Abstract

We explore statistical methods for financial asset management: non-parametric density estimation (with a focus on kernel density estimation and bandwidth selection), forecasting the distribution of option returns, systematic trading strategies built on density-derived statistics, and risk management (VaR/CVaR, GARCH, delta/vega hedging). We compare bandwidth selection approaches (rule-of-thumb, AMISE, LOOCV, MCMC, PIT, and a complexity-based criterion), extend to conditional densities (e.g., conditioned on volume), and apply methods across equities, commodities, indexes, and cryptocurrencies. We backtest strategies such as density-based expected-return/volatility filters and moving-average crossovers of *expected returns inferred from kernel densities*, including a variant with short positions.

---

## Table of Contents

- [1. Introduction](#1-introduction)  
- [2. Non-parametric Density Estimation](#2-non-parametric-density-estimation)  
  - [2.1 Classical: Rule of Thumb, AMISE, LOOCV](#21-classical-rule-of-thumb-amise-loocv)  
  - [2.2 MCMC Bandwidth](#22-mcmc-bandwidth)  
  - [2.3 PIT Bandwidth](#23-pit-bandwidth)  
  - [2.4 Complexity-based Bandwidth](#24-complexity-based-bandwidth)  
- [3. Conditional Densities](#3-conditional-densities)  
  - [3.1 Bootstrap Selection](#31-bootstrap-selection)  
  - [3.2 Threshold on Volume](#32-threshold-on-volume)  
- [4. Applications to Asset Classes](#4-applications-to-asset-classes)  
- [5. Option Returns & Hedging](#5-option-returns--hedging)  
  - [5.1 Future Option Value Distribution](#51-future-option-value-distribution)  
  - [5.2 Risk Measures & Vol Forecasting](#52-risk-measures--vol-forecasting)  
  - [5.3 Delta/Vega Replication](#53-deltavega-replication)  
- [6. Systematic Strategies](#6-systematic-strategies)  
  - [6.1 Backtest Framework](#61-backtest-framework)  
  - [6.2 KDE Expected Return + Volatility](#62-kde-expected-return--volatility)  
  - [6.3 Moving-Average Crossovers of KDE-Expectations](#63-moving-average-crossovers-of-kde-expectations)  
    - [6.3.1 Long-only](#631-long-only)  
    - [6.3.2 With Shorts](#632-with-shorts)  
- [7. Conclusion](#7-conclusion)  
- [8. References](#8-references)

---

## 1. Introduction

Financial decisions hinge on the distribution of returns. Non-parametric kernel density estimators (KDE) are flexible but depend critically on the *bandwidth* \(h\). We study several data-driven criteria for \(h\), then leverage these densities to:  
- Forecast option returns and risk (e.g., VaR on options), and  
- Build systematic strategies using density-derived statistics (expected returns, distributional signals).

---

## 2. Non-parametric Density Estimation

Given i.i.d. \(X_1,\dots,X_n\) with density \(f\), the kernel estimator is

\[
\hat f_h(x)=\frac1{nh}\sum_{i=1}^n K\!\left(\frac{x-X_i}{h}\right),\qquad
\hat F_h(x)=\frac1n\sum_{i=1}^n \kappa\!\left(\frac{x-X_i}{h}\right),
\]

with common kernels (Gaussian, Epanechnikov, Tophat, etc.).

### 2.1 Classical: Rule of Thumb, AMISE, LOOCV

- **Scott (1992)**: \(h\approx 3.5\,\hat\sigma\,n^{-1/3}\)  
- **Silverman (1986)**: \(h=0.9\min(\hat\sigma,\mathrm{IQR}/1.34)\,n^{-1/5}\)  
- **AMISE**: closed-form minimizer using \(R(K)\) and \(R(f'')\) yields \(h_{\text{AMISE}}\propto n^{-1/5}\).  
- **LOOCV**: maximize \(\sum_i \log \hat f_{-i}(x_i;h)\).

Empirically, Silverman and AMISE gave smooth fits on returns, while LOOCV’s optimum could oversmooth depending on the sample.

### 2.2 MCMC Bandwidth

We use Metropolis–Hastings to sample plausible \(h\) values by likelihood, then average accepted \(h\)’s. This produced realistic densities closely matching the data histogram, and we retained the posterior-mean \(h\) for later use.

### 2.3 PIT Bandwidth

We transform predictive CDF values via the **Probability Integral Transform** and pick \((h,w)\) (with exponential forgetting \(w\)) that best matches Uniform\([0,1]\) (via a distance \(d_\nu\)). The selected \(h\) coincides with well-calibrated predictive distributions.

### 2.4 Complexity-based Bandwidth

We maximize a complexity measure that is small under **underfitting** (KDE too close to Gaussian) and **overfitting** (KDE too close to empirical CDF), selecting \(h\) at the “sweet spot” between both extremes.

---

## 3. Conditional Densities

For covariate \(x\) (e.g., volume) and response \(y\) (return), the kernel conditional density is

\[
\hat f(y\mid x)=\frac{\hat g(x,y)}{\hat h(x)},\quad
\hat g(x,y)=\frac1{nab}\sum_{j=1}^n K\!\left(\frac{\lVert x-X_j\rVert}{a}\right)K\!\left(\frac{\lVert y-Y_j\rVert}{b}\right),\quad
\hat h(x)=\frac1{na}\sum_{j=1}^n K\!\left(\frac{\lVert x-X_j\rVert}{a}\right).
\]

### 3.1 Bootstrap Selection

We select \(a,b\) by minimizing bootstrap average error (integrated squared distance). Large bandwidths oversmooth; alternative divergences (e.g., KL) may improve sensitivity.

### 3.2 Threshold on Volume

Conditioning on volume above/below median shows similar shapes; Epanechnikov often fits better than Gaussian (less oversmoothing).

---

## 4. Applications to Asset Classes

We apply the toolkit to **stocks** (e.g., AAPL, MC.PA), **commodities** (GC=F), **indexes** (GSPC, NDX), and **cryptocurrencies** (BTC, ETH), observing model-suitability differences and tail sensitivity across assets.

---

## 5. Option Returns & Hedging

### 5.1 Future Option Value Distribution

From historical daily returns, we **resample** via bootstrap to simulate price paths \(S_t\) and obtain the distribution of option values at horizon \(k\). We also invert Black–Scholes to get **implied volatility** via Newton–Raphson.

### 5.2 Risk Measures & Vol Forecasting

- **VaR from KDE** at \(95\%\): worst expected loss not exceeded with prob. 95%.  
- **CVaR**: mean loss beyond VaR.  
- **GARCH(1,1)** forecasts volatility clusters to time-vary risk.

### 5.3 Delta/Vega Replication

We compute **Delta** and **Vega** (Black–Scholes) and solve a small optimization to achieve delta- and (optionally) vega-neutral portfolios using calls/puts/underlying.

---

## 6. Systematic Strategies

### 6.1 Backtest Framework

A Python `Backtest` class computes portfolio value under signals, with **transaction costs**, and reports **max drawdown**, **volatility**, and **total return** vs a benchmark; it also plots monthly return densities.

### 6.2 KDE Expected Return + Volatility

We estimate **expected return** and **volatility** from KDE over rolling windows; signals go **long** when expectation \(>\) 0 and volatility exceeds a threshold (to avoid noise), **flat/sell** otherwise.

**Illustrative example (index level):**  
- Strategy — lower MaxDD and Vol with higher Total Return vs benchmark in sample.  
- Benchmark — higher drawdowns and volatility.

### 6.3 Moving-Average Crossovers of KDE-Expectations

We compute **short** and **long** moving averages of the **KDE-based expected return** and trade on **crossovers**.

#### 6.3.1 Long-only
Across indices, large caps and crypto, the method generally **reduces drawdowns and volatility** while **increasing total return**; density plots show **shorter tails** and **higher medians** than benchmarks.

#### 6.3.2 With Shorts
On bearish crossovers, we allow **short exposure**. This further **improves downside capture** and often outperforms the benchmark, though explosive events can challenge short timing.

---

## 7. Conclusion

We provide a practical pipeline: **estimate KDEs with robust bandwidths**, extend to **conditional densities**, **forecast option return distributions**, and **trade** using density-derived statistics. The **MA-on-KDE-expectation** strategies are versatile across assets, typically improving the **risk-adjusted profile**. Future work: integrate **real-time data** and **ML** to refine density dynamics and decision rules.

---

## 8. References

1. Silverman, B. W. (1986). *Density Estimation for Statistics and Data Analysis*.  
2. Jones, M. C., Marron, J. S., & Sheather, S. J. (1996). A brief survey of bandwidth selection for density estimation. *JASA*, 91(433), 401–407.  
3. Tsybakov, A. B. (2003). *Introduction à l’estimation non paramétrique*.  
4. Zhang, X., Hyndman, R. J., & King, M. L. (2004). Bandwidth selection for multivariate KDE using MCMC.  
5. Harvey, A., & Oryshchenko, V. (2012). KDE for time series data. *IJF*, 28(1), 3–14.  
6. Garcin, M., Klein, J., & Laaribi, S. (2020). Time-varying kernels & COVID-19 chronology. *arXiv*.  
7. Garcin, M. (2023). Complexity measure, KDE, bandwidth, EMH. *arXiv*.  
8. Hyndman, R. J., Bashtannyk, D. M., & Grunwald, G. K. (1996). Conditional densities. *JCGS*, 5(4), 315–336.  
9. Bashtannyk, D. M., & Hyndman, R. J. (2001). Bandwidth for kernel conditional density. *CSDA*, 36(3), 279–298.  
10. Cao, J., et al. (2023). Gamma & vega hedging via deep RL. *Frontiers in AI*, 6, 1129370.  
11. Israelov, R., & Kelly, B. T. (2017). Forecasting the distribution of option returns. SSRN.

---

