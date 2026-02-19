# Appendix A2: Short-Rate Models (Vasicek / CIR / Hull–White)

---

## Introduction

Every rates desk has to answer a deceptively simple question: *how random is tomorrow's short rate?* The answer shapes everything from how you price a cap to how you hedge a swaption book. Short-rate models provide a mathematical framework for this randomness, specifying how the instantaneous interest rate evolves through time under a risk-neutral measure.

The choice of model affects three critical dimensions of rates practice. First, **curve dynamics**: how does the yield curve shift, twist, and change shape as time passes? A Vasicek model with high mean-reversion speed implies the front end is volatile while the back end is stable; a model with low mean-reversion gives more parallel shifts. Second, **option prices**: the volatility structure embedded in your model determines what you pay for convexity. A CIR model where volatility scales with $\sqrt{r}$ behaves very differently in a low-rate environment than a Gaussian model. Third, **risk sensitivities**: the Greeks you compute for hedging depend on model structure—one-factor models imply perfect correlation across tenors, which is a material simplification.

This appendix progresses through three canonical short-rate models, each representing an increase in sophistication:

- **Vasicek (Section 2)**: The simplest mean-reverting model—a Gaussian Ornstein-Uhlenbeck process. Closed-form bond prices and option formulas make it analytically tractable, but it allows negative rates and cannot match an arbitrary initial yield curve.

- **CIR (Section 3)**: Replaces constant volatility with $\sigma\sqrt{r}$, ensuring non-negative rates when the Feller condition holds. Still time-homogeneous, still cannot fit arbitrary curves, but captures the empirical observation that rate volatility increases with rate levels.

- **Hull-White (Section 4)**: Extends Vasicek with a time-dependent drift $\theta(t)$, enabling exact fit to any initial discount curve. This is the workhorse for practical derivatives pricing—it matches the market while retaining much of Vasicek's tractability.

We then cover **bond option pricing** (Section 5), including the Jamshidian decomposition that allows swaptions to be valued as portfolios of zero-coupon bond options. Section 6 presents the **Hull-White trinomial tree** construction essential for pricing American and Bermudan structures. Section 7 provides a **model comparison table**, and Section 8 addresses **calibration** to cap/floor and swaption markets. Throughout, we emphasize practical implementation alongside mathematical rigor.

**How to use this appendix (reading paths):**

- **If you’re here for Bermudans / callable structures:** start with Sections 4–6 (Hull–White + tree), then Sections 8 and 11 (calibration + QA), and refer to Section 5.4 (Jamshidian) as needed.
- **If you want the “why” behind desk model choices:** read Sections 1–4, then Section 7 (comparison) and Section 8 (practitioner workflow).
- **If measure changes are the stumbling block:** read Appendix A1 first (numeraires, forward/swap measures), then come back to Sections 1–2 here.
- **If you want the forward-rate viewpoint:** Section 9.6 connects short-rate models to HJM (see Appendix A3), and market models/LMM are in Appendix A4.

**Prerequisites:** Familiarity with stochastic calculus (Itô's lemma, Girsanov's theorem), risk-neutral pricing, and basic bond mathematics from earlier chapters.

---

## Conventions & Notation

| Symbol | Definition |
|--------|------------|
| $t \geq 0$ | Time in years |
| $T \gt t$ | Maturity |
| $r_t$ | Short rate at time $t$ (instantaneous continuously compounded rate) |
| $B(t)$ | Money-market account numeraire, $B(t) = \exp\left(\int_0^t r_u \\, du\right)$ |
| $Q$ | Risk-neutral (bank-account) measure |
| $W_t$ | Brownian motion under $Q$ |
| $P(t,T)$ | Zero-coupon bond price at $t$ for maturity $T$ |
| $D(t,T)$ | Money-market discount factor, $D(t,T) = \exp\left(-\int_t^T r_u \\, du\right)$ |
| $P^M(0,t)$ | Initial ("market") discount curve observed at time 0 |
| $f^M(0,t)$ | Market instantaneous forward rate, $f^M(0,t) = -\frac{\partial \ln P^M(0,t)}{\partial t}$ |

**Vasicek parameters:** $a \gt 0$ (mean-reversion speed, 1/year), $b$ (mean level, 1/year), $\sigma \gt 0$ (vol scale)

**CIR parameters:** $a \gt 0$, $b \geq 0$, $\sigma \gt 0$

**Hull–White (extended Vasicek) parameters:** $a \gt 0$, $\sigma \gt 0$, deterministic function $\theta(t)$

**Note:** Hull's notation in one-factor Hull–White uses $F(0,t)$ for the instantaneous forward at time 0.

### Key Relationships

**Money-market account** (standard numeraire for risk-neutral pricing):
$$B(t) = \exp\left(\int_0^t r_u \\, du\right)$$

**Risk-neutral pricing** for payoff $H$ at $T$:
$$V(t) = \mathbb{E}^Q\left[D(t,T) H \mid \mathcal{F}_t\right]$$

**Zero-coupon bond price:**
$$P(t,T) = \mathbb{E}^Q\left[D(t,T) \mid \mathcal{F}_t\right]$$

---

## 0. Setup

### Conventions used in this appendix

- One-factor short-rate diffusion models under a pricing (bank-account) measure $Q$
- Continuous-time, continuous compounding
- Curve-fitting language refers to matching the observed initial discount curve $P^M(0,T)$ (or equivalently `fM(0,t)`).

### Notation glossary (symbols + definitions)

| Symbol | Definition | Units |
|--------|------------|-------|
| $r_t$ | Short rate at time $t$ | 1/year |
| $B(t)$ | Money-market account numeraire | dimensionless |
| $D(t,T)$ | Money-market discount factor from $t$ to $T$ | dimensionless |
| $P(t,T)$ | ZCB price at $t$ for maturity $T$ | dimensionless |
| $Q$ | Bank-account (risk-neutral) measure | — |
| $W_t$ | Brownian motion under $Q$ | — |

---

## 1. Core Concepts (Definitions First)

### 1.1 Short Rate $r_t$ and Money-Market Account $B(t)$

**Formal Definition**

The short rate $r_t$ is the instantaneous continuously compounded riskless rate.

The money-market account (bank account numeraire) is:
$$B(t) = \exp\left(\int_0^t r_u \\, du\right)$$
which is the accumulation of investing at the short rate continuously.

**Intuition**

Over a small interval $[t, t + \Delta t]$, the bank account grows approximately by $r_t \Delta t$ in relative terms.

**Trading / Risk / Portfolio Practice**

- $r_t$ is not directly traded; it is a model state variable chosen so that the model implies ZCB prices and other rates
- $B(t)$ is the natural numeraire for discounting under $Q$ in short-rate models

### 1.2 Zero-Coupon Bond Price $P(t,T)$

**Formal Definition**

$P(t,T)$ is the time-$t$ price of a claim paying 1 at time $T$.

**Risk-Neutral Pricing Assumption (state clearly)**

Under an equivalent martingale measure associated with the bank-account numeraire, a payoff $H$ at time $T$ has time-$t$ price:
$$V(t) = \mathbb{E}^Q\left[D(t,T) H \mid \mathcal{F}_t\right], \quad D(t,T) = \exp\left(-\int_t^T r_u \\, du\right)$$

Taking $H = 1$ gives:
$$\boxed{P(t,T) = \mathbb{E}^Q\left[D(t,T) \mid \mathcal{F}_t\right]}$$

**Intuition**

A ZCB price is an average (under $Q$) of stochastic discounting driven by the path of the short rate.

**Trading / Risk / Portfolio Practice**

Even if you never trade ZCBs directly, they are the "atoms" of the curve: many rate products can be decomposed into claims on discount factors (e.g., caplets as portfolios of ZCB options in standard short-rate frameworks).

### 1.3 Risk-Neutral Measure and Numeraire Choice

**Formal Definition**

A pricing measure $Q$ is tied to a numeraire (here $B(t)$). Under this measure, discounted asset prices (by the numeraire) are martingales, enabling expectation-based pricing.

**Intuition**

You can price by "expected discounted payoff" once you specify:
1. What you discount with (numeraire), and
2. Under which measure you take expectations

**Trading / Risk / Portfolio Practice**

A recurring pitfall is mixing a real-world short-rate dynamics (for forecasting / scenario analysis) with a risk-neutral dynamics (for derivative pricing). Hull explicitly distinguishes real-world and risk-neutral processes for Vasicek/CIR via a market price of interest rate risk.

### 1.4 Affine Term Structure (What "Affine" Means)

**Formal Definition**

A term structure is affine (in the short rate) if the spot rate $R(t,T)$ is an affine function of $r_t$:
$$R(t,T) = \alpha(t,T) \\, r_t + \beta(t,T)$$

which is equivalent to an exponential-affine ZCB price:
$$\boxed{P(t,T) = A(t,T) \\, e^{-B(t,T) \\, r_t}}$$

**Intuition**

"Affine" means: once you know $r_t$, the whole curve $T \mapsto P(t,T)$ moves in a structured way, with $\log P$ linear in $r_t$.

**Trading / Risk / Portfolio Practice**

Affine structure is prized because it yields:
- Closed-form ZCB prices
- Often tractable option pricing (or at least efficient numerical methods)
- Simple risk sensitivities such as $\partial P / \partial r = -BP$ in these models

### 1.5 "Exogenous Term Structure" and Fitting the Initial Curve

**Formal Definition**

"Exogenous term structure" means the initial curve $P^M(0,T)$ (or `fM(0,t)`) is treated as an input from the market. The model is then constructed (or extended) so that model-implied bond prices match $P^M(0,T)$ at time 0.

**Key Fact About Basic (Time-Homogeneous) Short-Rate Models**

Time-homogeneous models (e.g., Vasicek/CIR with constant parameters) imply a specific functional family for $P(0,T)$. In practice this may fail to match the observed $P^M(0,T)$ even with parameter tuning, and the small number of parameters can prevent satisfactory calibration.

**Intuition**

In a basic one-factor model, the entire initial curve is "endogenous" — determined by a handful of parameters and the current state. Markets, however, provide a full curve across maturities, which is too rich to be matched exactly by a low-parameter model unless you add flexibility.

**Trading / Risk / Portfolio Practice**

Hull emphasizes that Vasicek/CIR are useful for simulating interest-rate behavior, but for derivatives valuation it is usually necessary to use a model that provides an exact fit to the current term structure.

The Hull–White extension introduces a deterministic time-dependent drift function precisely to achieve that curve fit (while keeping much of the tractability).

---

## 2. Vasicek Model (Gaussian OU Short Rate)

### 2.1 Model SDE

Under a pricing (risk-neutral) specification, the Vasicek short rate follows an Ornstein–Uhlenbeck (OU) process:

$$\boxed{dr_t = a(b - r_t) \\, dt + \sigma \\, dW_t}$$

This is the same structure as $dr_t = k(\theta - r_t) \\, dt + \sigma \\, dW_t$ used in other references, with $a \leftrightarrow k$ and $b \leftrightarrow \theta$.

**Real-World vs Risk-Neutral Note (Hull)**

Hull writes a real-world Vasicek with market price of risk $\lambda$, and shows the risk-neutral process has the same form with adjusted mean level $b^{\star}$ (notation in the source):
$$dr = a(b - r) \\, dt + \sigma \\, dz \quad \Rightarrow \quad dr = a(b^{\star} - r) \\, dt + \sigma \\, dz$$
where $b^{\star} = b + \lambda\sigma/a$.

### 2.2 Exact Solution for $r_t$ and Its Distribution

**Exact Solution (Integrating Factor Method)**

$$\boxed{r_t = r_0 e^{-at} + b(1 - e^{-at}) + \sigma \int_0^t e^{-a(t-u)} \\, dW_u}$$

**Conditional / Marginal Distribution**

The process is Gaussian with:

$$\boxed{\mathbb{E}[r_t] = r_0 e^{-at} + b(1 - e^{-at})}$$

$$\boxed{\text{Var}(r_t) = \frac{\sigma^2}{2a}(1 - e^{-2at})}$$

**Negative Rate Implication**

Because the distribution is normal, negative rates can occur with positive probability. This is explicitly noted as a consequence of the normal distribution of the short rate.

### 2.3 Bond Pricing: Affine Form and $A(t,T)$, $B(t,T)$

**Affine (Exponential-Affine) Bond Price**

$$P(t,T) = A(t,T) \exp(-B(t,T) \\, r_t)$$

**Closed-Form $B(t,T)$**

$$\boxed{B(t,T) = \frac{1 - e^{-a(T-t)}}{a}}$$

**Closed-Form $A(t,T)$ (Hull's Expression)**

$$\boxed{A(t,T) = \exp\left(\frac{(B(t,T) - T + t)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B(t,T)^2}{4a}\right)}$$

### 2.4 Practical Interpretation: What It Fits Well and What It Fails At

**Fits Well (Why Used)**

- **Tractability:** Gaussian OU dynamics yield closed-form expressions for ZCB prices (and, in many treatments, tractable option pricing). The model is listed as having analytical bond and bond option prices in the comparative summary table of short-rate models.
- **Mean reversion:** parameter $a$ controls the speed at which $r_t$ is pulled toward $b$.

**Fails / Limitations (As Supported)**

- **Negative rates:** normality allows $r_t \lt 0$
- **Initial curve fit:** time-homogeneous short-rate models generally do not match the observed initial term structure, and the limited number of parameters can prevent satisfactory calibration to market curves

Hull specifically notes the Vasicek/CIR models do not provide an exact fit to the current term structure, which motivates using models (like Hull–White) that do.

### 2.5 Unit Checks and Limiting Cases

**Units**
- $r$ is $1/\text{year}$
- $B(t,T)$ has units of years, so $B(t,T) \\, r_t$ is dimensionless
- $A(t,T)$ is dimensionless, and $P(t,T) \in (0, \infty)$ is dimensionless

**Limiting Cases**

| Limit | Result |
|-------|--------|
| $T \to t$ | $B(t,T) \to 0$, $A(t,T) \to 1$, so $P(t,t) = 1$ (boundary condition) |
| $\sigma \to 0$ | Stochastic integral disappears; $r_t$ becomes deterministic (pure mean reversion) |
| $a \to 0$ | $B(t,T) \to (T-t)$ by the expansion $1 - e^{-a\Delta} \sim a\Delta$ |

### 2.6 Real-World vs Risk-Neutral Dynamics and Estimation

**The Fundamental Distinction**

Brigo and Mercurio emphasize a critical point: "The pioneering approach proposed by Vasicek (1977) was based on defining the instantaneous-spot-rate dynamics under the real-world measure." For pricing, we use the risk-neutral measure $Q$; for historical estimation and forecasting, we need the physical measure $Q_0$.

**Real-World (Physical) Dynamics**

Under the physical measure $Q_0$, the Vasicek process can be written as:

$$\boxed{dr(t) = [k\theta - (k + \lambda\sigma)r(t)] \\, dt + \sigma \\, dW^0(t)}$$

where $\lambda$ is a new parameter contributing to the market price of risk. This has the same Gaussian structure as the risk-neutral dynamics but with different drift parameters.

**Connection to Risk-Neutral Dynamics**

The market price of risk process $\lambda(t) = \lambda r(t)$ (proportional to the short rate) allows both dynamics to remain tractable. The Girsanov change of measure:

$$\left.\frac{dQ}{dQ_0}\right|_{\mathcal{F}_t} = \exp\left(-\frac{1}{2}\int_0^t \lambda^2 r(s)^2 \\, ds + \int_0^t \lambda r(s) \\, dW^0(s)\right)$$

**Why Both Measures Matter**

| Measure | Use | What We Observe |
|---------|-----|-----------------|
| Risk-neutral $Q$ | Derivative pricing | Market prices |
| Physical $Q_0$ | Historical estimation, VaR, forecasting | Historical rate time series |

As Brigo and Mercurio note: "Data are collected in the real world, and their statistical properties characterize the distribution of our interest-rate process under the objective measure."

**Maximum Likelihood Estimation (Brigo & Mercurio)**

Given daily observations $r_0, r_1, \ldots, r_n$ with time step $\delta$, define transformed parameters:

$$\alpha := e^{-a\delta}, \quad \beta := b/a, \quad V^2 := \frac{\sigma^2}{2a}(1 - e^{-2a\delta})$$

The maximum likelihood estimators are:

$$\boxed{\hat{\alpha} = \frac{n\sum_{i=1}^n r_i r_{i-1} - \sum_{i=1}^n r_i \sum_{i=1}^n r_{i-1}}{n\sum_{i=1}^n r_{i-1}^2 - \left(\sum_{i=1}^n r_{i-1}\right)^2}}$$

$$\boxed{\hat{\beta} = \frac{\sum_{i=1}^n [r_i - \hat{\alpha}r_{i-1}]}{n(1-\hat{\alpha})}}$$

$$\boxed{\widehat{V^2} = \frac{1}{n}\sum_{i=1}^n [r_i - \hat{\alpha}r_{i-1} - \hat{\beta}(1-\hat{\alpha})]^2}$$

**Practical Workflow**

1. Estimate $\sigma$ from historical data using ML (diffusion coefficient is the same under both measures)
2. Calibrate $a$ and $\theta$ to market derivative prices under $Q$
3. Or: use historical estimates as initial guesses, then optimize to match market prices

---

## 3. CIR Model (Square-Root Diffusion)

### 3.1 Model SDE

The CIR short rate follows:

$$\boxed{dr_t = a(b - r_t) \\, dt + \sigma\sqrt{r_t} \\, dW_t}$$

Hull notes a common specification for the market price of interest rate risk in CIR, $\lambda = \kappa\sqrt{r}$, and derives that the risk-neutral process remains CIR-form with adjusted parameters.

### 3.2 Positivity Discussion

**Non-Negativity**

Hull states that (unlike Vasicek) CIR does not allow negative short rates.

**Strict Positivity Condition (Feller-Type)**

Hull: if $2ab \geq \sigma^2$, then $r(t)$ is never zero; otherwise it can touch zero.

Interest Rate Modeling explicitly links the classical strict positivity requirement for CIR to $2\kappa\vartheta \geq \sigma^2$ (same structure with different notation).

### 3.3 Transition Distribution: Non-Central Chi-Squared

Unlike Vasicek where $r(t)$ is normally distributed, the CIR process has a **non-central chi-squared** transition distribution. This is crucial for understanding simulation and option pricing.

**Transition Density**

Given $r(s) = r_s$, the distribution of $r(t)$ for $t \gt s$ is:

$$r(t) \sim \frac{\sigma^2(1 - e^{-a(t-s)})}{4a} \cdot \chi^2_\nu(\lambda)$$

where $\chi^2_\nu(\lambda)$ denotes the non-central chi-squared distribution with:

- **Degrees of freedom:** $\nu = \frac{4ab}{\sigma^2}$
- **Non-centrality parameter:** $\lambda = \frac{4a \cdot e^{-a(t-s)}}{\sigma^2(1 - e^{-a(t-s)})} \cdot r_s$

**Connection to Feller Condition**

The degrees of freedom $\nu = 4ab/\sigma^2$:
- If $\nu \geq 2$ (i.e., $2ab \geq \sigma^2$): the process is strictly positive
- If $\nu \lt 2$: the process can reach zero but immediately reflects

**Conditional Moments**

$$\mathbb{E}[r(t) \mid r(s)] = r(s) e^{-a(t-s)} + b(1 - e^{-a(t-s)})$$

$$\text{Var}[r(t) \mid r(s)] = r(s) \frac{\sigma^2}{a}(e^{-a(t-s)} - e^{-2a(t-s)}) + \frac{b\sigma^2}{2a}(1 - e^{-a(t-s)})^2$$

### 3.3.1 Simulating CIR Paths: Methods and Pitfalls

Monte Carlo simulation of CIR is essential for PFE calculations, XVA, and risk management. However, naive discretization fails near zero. This subsection covers the three main approaches.

**Method 1: Exact Simulation via Non-Central Chi-Squared**

The most accurate approach uses the exact transition density. Given $r(t_i)$, sample $r(t_{i+1})$ as:

$$r(t_{i+1}) = c \cdot \chi^2_\nu(\lambda)$$

where $c = \frac{\sigma^2(1 - e^{-a\Delta t})}{4a}$, $\nu = \frac{4ab}{\sigma^2}$, and $\lambda = \frac{4ae^{-a\Delta t}}{\sigma^2(1 - e^{-a\Delta t})} r(t_i)$.

**Implementation:** Use the inverse CDF of the non-central chi-squared distribution (available in most statistical libraries).

**Drawback:** Non-central chi-squared inversion is computationally expensive, especially when the non-centrality parameter $\lambda$ is large.

**Method 2: Euler Discretization with Reflection**

The simplest approach discretizes the SDE directly:

$$r(t_{i+1}) = r(t_i) + a(b - r(t_i))\Delta t + \sigma\sqrt{r(t_i)}\sqrt{\Delta t} \\, Z_i$$

where $Z_i \sim N(0,1)$.

**Problem:** This can produce $r(t_{i+1}) \lt 0$, which breaks the $\sqrt{r}$ term at the next step.

**Fix (Reflection):** Replace negative values with their absolute value: $r(t_{i+1}) \leftarrow |r(t_{i+1})|$.

**Warning:** Reflection introduces bias, especially when the Feller condition is violated ($\nu \lt 2$). The bias can be significant for long simulation horizons or near-zero rate environments.

**Method 3: Quadratic-Exponential (QE) Scheme (Andersen)**

The QE scheme, developed by Andersen (2008), achieves near-exact accuracy with computational efficiency close to Euler. It uses moment-matching with different sampling distributions depending on the current rate level.

**Key Idea:** The scheme switches between two sampling methods based on $\psi = \frac{s^2}{m^2}$ where $m = \mathbb{E}[r(t_{i+1}) | r(t_i)]$ and $s^2 = \text{Var}[r(t_{i+1}) | r(t_i)]$.

**Case 1 ($\psi \leq \psi^{\star}$, typically $\psi^{\star} = 1.5$):** Use a quadratic form:

$$r(t_{i+1}) = a(b + \sqrt{Z_V})^2$$

where $b$ and $a$ are chosen to match the first two moments.

**Case 2 ($\psi \gt \psi^{\star}$):** Use an exponential approximation:

$$r(t_{i+1}) = \begin{cases} 0 & \text{if } U \leq p \\ \beta^{-1}\ln\left(\frac{1-p}{1-U}\right) & \text{if } U \gt p \end{cases}$$

where $U \sim \text{Uniform}(0,1)$, and $p$, $\beta$ are chosen to match moments.

> **Desk Reality: Simulation Method Selection**
>
> In production systems for PFE and XVA, the QE scheme is the standard choice. It combines the accuracy of exact simulation with computational speed close to Euler. Always verify the Feller condition before choosing your method:
> - **$\nu \geq 2$ (Feller satisfied):** Any method works well; QE preferred for speed
> - **$\nu \lt 2$ (Feller violated):** Avoid Euler with reflection; use QE or exact simulation
>
> For quick prototyping, Euler with reflection is acceptable but should be replaced for production use.

### 3.4 Bond Pricing: Affine Form and $A(t,T)$, $B(t,T)$

**Affine Bond Price**

$$P(t,T) = A(t,T) \\, e^{-B(t,T) \\, r(t)}$$

**Closed-Form $B(t,T)$ and $A(t,T)$ (Hull)**

Define:
$$\boxed{\gamma = \sqrt{a^2 + 2\sigma^2}}$$

Then:

$$\boxed{B(t,T) = \frac{2(e^{\gamma(T-t)} - 1)}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}}$$

$$\boxed{A(t,T) = \left[\frac{2\gamma \exp\left(\frac{(a+\gamma)(T-t)}{2}\right)}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}\right]^{2ab/\sigma^2}}$$

**Useful Sensitivity Identity**

For both Vasicek and CIR:
$$\boxed{\frac{\partial P(t,T)}{\partial r(t)} = -B(t,T) \\, P(t,T)}$$

### 3.5 Practical Interpretation

**Level-Dependent Volatility**

Hull: the standard deviation of the change in the short rate in a small time interval is proportional to $\sqrt{r}$, so volatility increases with the level of rates.

**Benefits vs Vasicek**

Positivity / non-negativity is a key advantage compared with Gaussian models.

**Limitations (As Supported)**

As with Vasicek, the basic time-homogeneous CIR structure generally will not match the observed initial curve exactly; exact initial-curve fit motivates deterministic-shift extensions (CIR++ / shifted CIR), which are treated as extensions in the term-structure modeling literature.

### 3.6 Unit/Limiting Checks

**Units**

Same affine check: $B(t,T)$ has units of time; $Br$ is dimensionless; $A$ dimensionless.

**Limiting Checks**

| Limit | Result |
|-------|--------|
| $T \to t$ | $B(t,T) \to 0$ and $P(t,t) = 1$ (consistent with ZCB payoff) |
| $\sigma \to 0$ | Diffusion vanishes; model becomes deterministic mean reversion |

---

## 3.7 The Ho-Lee Model: Time-Dependent Drift Without Mean Reversion

Before examining Hull-White, it is instructive to consider the simpler Ho-Lee model (1986), which introduces time-dependent drift but lacks mean reversion. This model represents a conceptual bridge between time-homogeneous models (Vasicek, CIR) and the full Hull-White extension.

### 3.7.1 Model SDE

The Ho-Lee dynamics under the risk-neutral measure are:

$$\boxed{dr = \lambda(t) \\, dt + \sigma \\, dW}$$

where $\lambda(t)$ is a deterministic drift function and $\sigma$ is constant volatility. As Tuckman notes: "In contrast to Model 2 [constant drift], the drift here depends on time. In other words, the drift of the process may change from date to date."

**Key Observation:** There is no $-ar$ term pulling rates back to a long-run mean. The short rate follows a random walk with time-varying drift.

### 3.7.2 Fitting the Initial Term Structure

The flexibility of the Ho-Lee model comes from choosing $\lambda(t)$ to match market bond prices. Tuckman describes the procedure:

> "The free parameters $\lambda_1$ and $\lambda_2$ may be used to match the prices of bonds with fixed cash flows... set $r_0$ equal to the one-month rate. Next, obtain a monthly spot rate curve from traded bond prices... Then find $\lambda_1$ such that the model produces a two-month spot rate equal to that of the initial spot rate curve."

This iterative fitting ensures the model reproduces any given initial term structure.

### 3.7.3 Bond Pricing

The bond price in Ho-Lee has the affine form:

$$P(t,T) = A(t,T) \\, e^{-B(t,T) \\, r(t)}$$

with:
$$B(t,T) = T - t$$

This is the $a \to 0$ limit of Vasicek's $B(t,T) = \frac{1-e^{-a(T-t)}}{a}$.

### 3.7.4 Volatility Structure and Limitations

**Constant Forward Rate Volatility:** In Ho-Lee, the instantaneous forward rate volatility is constant across all maturities:
$$\sigma_f(t,T) = \sigma$$

This implies parallel shifts of the yield curve—all forward rates move together with perfect correlation.

**No Mean Reversion:** Without the $-ar$ term, short rates can drift arbitrarily far from any central value. As Tuckman warns: "The rate curves resulting from this model match all the rates that are input into the model. Just as adding a constant drift to Model 1 does not affect the shape of the term structure of volatility nor the parallel shift characteristic of the model, making the drift time-dependent also does not change these features."

**When Ho-Lee is Appropriate:**
- Short-dated options where mean reversion is less important
- As a limiting case for understanding Hull-White
- When only the initial curve fit matters, not long-horizon dynamics

### 3.7.5 Relationship to Hull-White

Hull-White can be viewed as Ho-Lee plus mean reversion:

| Feature | Ho-Lee | Hull-White |
|---------|--------|------------|
| Drift | $\lambda(t)$ | $\theta(t) - ar$ |
| Mean reversion | None | Speed $a$ |
| Forward vol structure | Constant | Exponentially decaying |
| $B(t,T)$ | $T - t$ | $\frac{1-e^{-a(T-t)}}{a}$ |

As $a \to 0$ in Hull-White, we recover Ho-Lee (with $\sigma_p = \sigma(S-T)\sqrt{T}$).

---

## 4. Hull–White (Extended Vasicek / Time-Dependent Drift) and "Fitting the Initial Curve"

### 4.1 Model Form and Relationship to Vasicek

**Hull–White One-Factor (Extended Vasicek) Model**

$$\boxed{dr(t) = [\theta(t) - a \\, r(t)] \\, dt + \sigma \\, dz}$$

where $\theta(t)$ is a deterministic function.

**Relationship to Vasicek**

Hull notes this is Vasicek with a time-dependent mean-reversion level:
$$dr(t) = a \\, [b(t) - r(t)] \\, dt + \sigma \\, dz, \quad b(t) = \frac{\theta(t)}{a}$$

### 4.2 How theta(t) Is Chosen to Fit Today's Curve P^M(0,T)

This is the exogenous term structure idea: the initial curve is taken from the market and built into the model.

**Step 1 (Input): Market Curve $P^M(0,t)$ and Its Forward Rates**

Assume the market discount curve $t \mapsto P^M(0,t)$ is given and sufficiently smooth, and define the market instantaneous forward rates:

$$f^M(0,t) = -\frac{\partial \ln P^M(0,t)}{\partial t}$$

**Step 2 (Choose $a, \sigma$; Then set `theta(t)`)**

Hull provides the specific function:

$$\boxed{\theta(t) = \frac{\partial F(0,t)}{\partial t} + a \\, F(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})}$$

where $F(0,t)$ is the instantaneous forward rate at time 0 for maturity $t$.

**Step 3 (Interpretation: What Is Exogenous vs Endogenous)**

| Type | Description |
|------|-------------|
| **Exogenous** | The entire initial curve $P^M(0,T)$ (or $F(0,t)$) is an input |
| **Endogenous** | Future randomness of $r_t$ (and therefore future $P(t,T)$) is generated by the diffusion with parameters $(a, \sigma)$ and deterministic $\theta(t)$ |

This construction is explicitly motivated by the observed "poor fitting of the initial term structure" by time-homogeneous models and the need to impose the initial curve.

### 4.3 Bond Pricing and Affine Form Under Hull–White

**Affine Bond Price**

$$P(t,T) = A(t,T) \exp(-B(t,T) \\, r(t))$$

**$B(t,T)$**

$$\boxed{B(t,T) = \frac{1 - e^{-a(T-t)}}{a}}$$

**$A(t,T)$ Explicitly Uses the Initial Curve**

Hull gives:

$$\boxed{\ln A(t,T) = \ln\frac{P(0,T)}{P(0,t)} + B(t,T) \\, F(0,t) - \frac{\sigma^2}{4a^3}(e^{-aT} - e^{-at})^2(e^{2at} - 1)}$$

**Key "Fit" Message**

Because $A(t,T)$ is built from $P(0,T)/P(0,t)$, the model is designed to be consistent with the initial curve used in the construction.

### 4.4 Practical Interpretation: What It Fits and Why; What It Doesn't

**What It Fits (And Why)**

- **Exact fit to initial curve:** the time-dependent drift is chosen precisely to incorporate the observed initial discount curve into the model. This addresses the limitation of time-homogeneous models that do not match the observed term structure.
- **Tractability:** in one-factor Gaussian exogenous term structure models (including Hull–White), ZCB bonds and (in many setups) options on ZCBs can be priced explicitly, which then supports pricing of caps/floors as portfolios of ZCB options and swaptions via Jamshidian decomposition.

**What It Doesn't (As Supported)**

- **Negative rates:** Hull–White is Gaussian; short rates are normally distributed and can become negative.
- **One-factor limitation:** a one-factor short-rate model implies perfect correlation between rates of different maturities, which is cited as a limitation for concrete pricing problems; these models are noted as still used for risk management.
- **Fitting volatility term structure by time-dependent $a(t), \sigma(t)$:** making more parameters time-dependent can fit more market features but is described as potentially dangerous due to unstable parameter estimation and the need for ad-hoc functional forms; the chapter motivation is to use only a deterministic shift/drift to fit the yield curve.

**Multi-Curve Considerations**

The models in this appendix are formulated in a single-curve setting: one discount curve $P(0,T)$ drives both discounting and rate projection. In modern practice, OIS discounting is standard for collateralized derivatives, while projection curves (SOFR, €STR) determine floating payments.

For Hull-White calibration in a multi-curve world, the typical approach is:
- Use the OIS curve as the discount curve $P^M(0,T)$ that determines $\theta(t)$
- Price caps/floors using OIS-discounted expectations of SOFR-linked payoffs
- The short rate $r_t$ in the model corresponds to the overnight rate (not a term rate)

The single-curve formulas in this appendix apply directly when the discount and projection curves coincide, or as approximations when basis is small. For precise multi-curve treatment with significant basis, see the multi-curve framework in Chapter 19.

---

### 4.5 The CIR++ Model: Fitting the Curve with Non-Negative Rates

**Motivation: Combining CIR's Positivity with Curve Fitting**

The Hull-White model fits the initial curve exactly but allows negative rates. The basic CIR model guarantees positive rates but cannot match an arbitrary initial curve. The CIR++ model, introduced by Brigo and Mercurio, combines the best of both: it preserves CIR's non-negativity while fitting the observed term structure exactly.

**The Deterministic-Shift Construction**

Brigo and Mercurio define the CIR++ model as a deterministic shift of the basic CIR process:

$$\boxed{r(t) = x(t) + \varphi(t)}$$

where $x(t)$ follows the standard CIR dynamics:

$$dx(t) = k(\theta - x(t)) \\, dt + \sigma \sqrt{x(t)} \\, dW(t), \quad x(0) = x_0$$

with parameters satisfying the Feller condition $2k\theta \gt \sigma^2$ to ensure $x(t) \gt 0$ for all $t$.

**The Shift Function `varphi(t)`**

To fit the market curve $P^M(0,T)$ exactly, the shift function is determined by:

$$\boxed{\varphi^{CIR}(t; \alpha) = f^M(0,t) - f^{CIR}(0,t; \alpha)}$$

where $f^M(0,t) = -\frac{\partial}{\partial t} \ln P^M(0,t)$ is the market instantaneous forward rate, and $f^{CIR}(0,t; \alpha)$ is the forward rate implied by the basic CIR model with parameter vector $\alpha = (k, \theta, \sigma, x_0)$.

Brigo and Mercurio give the explicit CIR forward rate:

$$f^{CIR}(0,t; \alpha) = \frac{2k\theta(e^{th} - 1)}{2h + (k+h)(e^{th} - 1)} + x_0 \frac{4h^2 e^{th}}{[2h + (k+h)(e^{th} - 1)]^2}$$

where $h = \sqrt{k^2 + 2\sigma^2}$.

**Bond Pricing Under CIR++**

The price at time $t$ of a zero-coupon bond maturing at $T$ is:

$$P(t,T) = \bar{A}(t,T) \\, e^{-B(t,T) r(t)}$$

where $B(t,T)$ is the same as in basic CIR (eq. 3.25), and:

$$\bar{A}(t,T) = \frac{P^M(0,T) \\, A(0,t) \\, e^{-B(0,t) x_0}}{P^M(0,t) \\, A(0,T) \\, e^{-B(0,T) x_0}} \cdot A(t,T) \\, e^{B(t,T) \varphi^{CIR}(t;\alpha)}$$

This formula shows how the market curve $P^M(0,T)$ enters directly into the bond pricing, ensuring exact fit at $t=0$.

**Rate Positivity and Fitting Quality**

A key practical consideration is whether $r(t) = x(t) + \varphi(t)$ remains positive. Since $x(t) \gt 0$ (under the Feller condition), positivity of $r(t)$ requires:

$$\varphi(t) \gt -x(t) \quad \text{for all } t$$

In practice, this is approximately ensured if:

$$\varphi(t) = f^M(0,t) - f^{CIR}(0,t; \alpha) \gt 0$$

which holds when the market curve lies above the basic CIR-implied curve. If the market curve is very low (negative forward rates in some tenors), $\varphi(t)$ may become sufficiently negative to push $r(t)$ negative, defeating the purpose. Brigo and Mercurio note this as a practical calibration consideration.

**Comparison: Hull-White vs. CIR++**

| Feature | Hull-White | CIR++ |
|---------|-----------|-------|
| Curve fitting | Exact | Exact |
| Rate positivity | No (Gaussian) | Yes (if $\varphi(t)$ not too negative) |
| Analytical tractability | Full (bond prices, options) | Partial (bond prices; options via numerical methods) |
| Option pricing | Closed-form ZBO | Chi-squared distribution, more complex |
| Typical use | Rates desks, general pricing | Settings where positivity matters |

**When to Use CIR++**

CIR++ is valuable when:
- Rate positivity is a hard constraint (e.g., some risk systems)
- The curve is not too flat or inverted (avoiding deeply negative $\varphi$)
- You can accept more complex option pricing (no simple Black-like formula)

For most practical purposes in a low-but-positive rate environment, Hull-White remains the workhorse due to its full analytical tractability.

---

### 4.6 The Black-Karasinski Model: Lognormal Short Rates with Mean Reversion

The Black-Karasinski (1991) model addresses the negative-rate problem through a different mechanism than CIR: it models the *logarithm* of the short rate as a Gaussian process, ensuring the short rate itself is always positive. This model is the "lognormal counterpart" to Hull-White and remains popular among practitioners who require strict rate positivity.

**4.6.1 Model SDE**

Black and Karasinski assumed that the logarithm of the instantaneous short rate evolves under the risk-neutral measure as:

$$\boxed{d\ln r(t) = [\theta(t) - a\ln r(t)] \\, dt + \sigma \\, dW(t)}$$

where $\theta(t)$ is a deterministic function chosen to fit the initial term structure, $a \gt 0$ is the mean-reversion speed, and $\sigma \gt 0$ is the volatility of the log-rate. As Brigo and Mercurio note: "Black and Karasinski assumed that the instantaneous short rate process evolves as the exponential of an Ornstein-Uhlenbeck process."

**Equivalent Form via Itô's Lemma:**

Applying Itô's lemma to $r(t) = \exp(\ln r(t))$:

$$dr(t) = r(t)\left[\theta(t) + \frac{\sigma^2}{2} - a\ln r(t)\right] dt + \sigma r(t) \\, dW(t)$$

**Key Property:** The diffusion coefficient is $\sigma r(t)$, meaning volatility is proportional to the rate level (similar to CIR, but with a different functional form).

**4.6.2 Distribution and Positivity**

**Conditional Distribution:** Given $r(s)$, the conditional distribution of $r(t)$ for $t \gt s$ is:

$$r(t) = \exp\left\\{\ln r(s) \\, e^{-a(t-s)} + \int_s^t e^{-a(t-u)}\theta(u) \\, du + \sigma\int_s^t e^{-a(t-u)} \\, dW(u)\right\\}$$

The stochastic integral is Gaussian, so $\ln r(t) | \mathcal{F}_s$ is normally distributed. Therefore:

$$\boxed{r(t) | \mathcal{F}_s \text{ is lognormally distributed}}$$

**Strict Positivity:** Since the exponential of any real number is positive, $r(t) \gt 0$ always. Unlike CIR, there is no Feller-type condition—positivity is automatic.

**4.6.3 The Analytical Tractability Trade-off**

The Black-Karasinski model has a critical limitation: **no closed-form bond prices**.

Brigo and Mercurio explain: "The Black-Karasinski (1991) model is not analytically tractable. This renders the model calibration to market data more burdensome than in the Hull and White (1990) Gaussian model, since no analytical formulas for bonds are available."

**Consequences for Pricing:**
- **ZCB pricing:** Requires a tree or PDE solver at each evaluation point
- **Bond options:** Must be priced numerically (no Jamshidian decomposition shortcut)
- **Simulation:** To simulate forward rates, one must build a tree for each simulated path

As Brigo and Mercurio note: "When we have simulated $r(1y)$, we need to compute $P(1y, 5y)$ numerically for each simulated realization of $r$. In practice, we need to use a tree for each simulated $r$."

**4.6.4 Tree Construction**

Since analytical formulas are unavailable, Black-Karasinski is implemented via trinomial trees. The approach mirrors Hull-White tree construction (Section 6), with one modification:

$$r(t) = e^{\alpha(t) + x(t)}$$

where $x(t)$ follows the zero-mean OU process used in the Hull-White tree, and $\alpha(t)$ is the deterministic shift that fits the initial curve.

**Procedure:**
1. Build a trinomial tree for the $x$ process (as in Hull-White Stage 1)
2. At each node $(i,j)$, the short rate is $r_{i,j} = \exp(\alpha_i + x_{i,j})$
3. Determine $\alpha_i$ iteratively to match market discount factors

**4.6.5 Infinite Expectation Problem**

A theoretical concern with lognormal short-rate models: the expected value of the money-market account is infinite for any maturity. This arises because the lognormal distribution has a fat right tail.

Brigo and Mercurio note: "The expected value of the money-market account is infinite no matter which maturity is considered, as a consequence of the lognormal distribution of $r$. As a consequence, the price of a Eurodollar future is infinite, too."

**Practical Mitigation:** When using trees, one deals with a finite number of states, so expectations are finite. This is not a major issue in practice.

**4.6.6 Comparison: Hull-White vs Black-Karasinski**

| Feature | Hull-White | Black-Karasinski |
|---------|-----------|------------------|
| Short rate distribution | Normal (Gaussian) | Lognormal |
| Rate positivity | No (can be negative) | Yes (always positive) |
| ZCB pricing | Closed-form | Numerical (tree/PDE) |
| Bond option pricing | Closed-form (Jamshidian) | Numerical |
| Calibration speed | Fast (analytical) | Slow (tree for each bond) |
| Volatility structure | Constant absolute vol | Constant percentage vol |
| Typical use | General-purpose, fast calibration | When positivity required and numerical cost acceptable |

> **Desk Reality: When to Use Black-Karasinski**
>
> Black-Karasinski is chosen when:
> - **Rate positivity is a hard requirement** (e.g., some risk systems reject negative rates)
> - **Calibration to swaption volatilities is important** (BK often fits the swaption surface well due to its flexibility)
> - **Computational cost is acceptable** (you have time for tree-based pricing)
>
> In practice, many desks have moved away from BK due to:
> - The negative rate environment (where positivity isn't always desirable)
> - The computational burden for XVA and PFE calculations
> - The availability of CIR++ as an alternative positive-rate model with partial tractability
>
> That said, BK remains popular for Bermudan swaption pricing where its good fit to the swaption surface outweighs the computational cost.

---

### 4.7 Preview: Two-Factor Extensions (G2++)

One-factor models, by construction, imply perfect correlation between rates of all maturities. This is a significant limitation for products whose value depends on the relative movement of different points on the curve (e.g., spread options, CMS spread caps). Two-factor models address this by introducing a second source of randomness.

**4.7.1 The G2++ Model (Brigo-Mercurio)**

The G2++ model extends the one-factor Hull-White framework by writing the short rate as the sum of two correlated Gaussian factors plus a deterministic shift:

$$\boxed{r(t) = x(t) + y(t) + \varphi(t)}$$

where:

$$dx(t) = -a \\, x(t) \\, dt + \sigma \\, dW_1(t), \quad x(0) = 0$$
$$dy(t) = -b \\, y(t) \\, dt + \eta \\, dW_2(t), \quad y(0) = 0$$

with $dW_1(t) \\, dW_2(t) = \rho \\, dt$ (instantaneous correlation).

**Parameters:**
- $a, b \gt 0$: mean-reversion speeds for each factor
- $\sigma, \eta \gt 0$: volatilities of each factor
- $\rho \in [-1, 1]$: correlation between factors
- $\varphi(t)$: deterministic shift to fit the initial curve

**4.7.2 Key Properties**

**Curve Fitting:** The deterministic function $\varphi(t)$ is chosen to fit the initial term structure exactly:

$$\varphi(T) = f^M(0,T) + \frac{\sigma^2}{2a^2}(1-e^{-aT})^2 + \frac{\eta^2}{2b^2}(1-e^{-bT})^2 + \rho\frac{\sigma\eta}{ab}(1-e^{-aT})(1-e^{-bT})$$

**Closed-Form Bond Prices:** Despite having two factors, G2++ retains analytical tractability:

$$P(t,T) = \frac{P^M(0,T)}{P^M(0,t)} \exp\left\\{\mathcal{A}(t,T) - \frac{1-e^{-a(T-t)}}{a}x(t) - \frac{1-e^{-b(T-t)}}{b}y(t)\right\\}$$

**Imperfect Correlation:** The key advantage—rates at different maturities are not perfectly correlated. The correlation structure is determined by the parameters $(a, b, \sigma, \eta, \rho)$.

**4.7.3 Why Two Factors Matter**

| Product | One-Factor Limitation | Two-Factor Advantage |
|---------|----------------------|---------------------|
| CMS spread options | Cannot capture spread dynamics | Spread driven by factor correlation |
| Curve steepeners/flatteners | All points move together | Short and long ends can diverge |
| Swaption cube calibration | One-factor can't fit full cube | Five parameters enable better fit |
| Bermudan swaptions | May misprice exercise boundary | More realistic curve dynamics |

Brigo and Mercurio note: "Another consequence of the presence of two factors is that the actual variability of market rates is described in a better way: Among other improvements, a non-perfect correlation between rates of different maturities is introduced."

**4.7.4 When to Use Two-Factor Models**

| Use Case | One-Factor OK? | Two-Factor Needed? |
|----------|---------------|-------------------|
| Vanilla caps/floors | Yes | No |
| European swaptions (ATM) | Usually | Sometimes |
| Bermudan swaptions | Often | Preferred |
| CMS products | No | Yes |
| Spread options | No | Yes |
| PFE/XVA | Often | Preferred for accuracy |

> **Desk Reality: The One-Factor vs Two-Factor Decision**
>
> Most rates desks use one-factor Hull-White as their workhorse due to speed and simplicity. Two-factor models (G2++ or equivalently the Hull-White two-factor model) are deployed selectively for:
> - Products with explicit curve-shape dependence
> - High-value Bermudan swaption books where calibration accuracy justifies the cost
> - XVA calculations where curve dynamics materially affect PFE
>
> The computational cost of two-factor models is roughly 2-3x that of one-factor for tree-based pricing, and the calibration is more complex (5 parameters vs 2). This is manageable for pricing individual trades but can become expensive for portfolio-level calculations.

**Note:** Full treatment of two-factor models and the connection to HJM is provided in Appendix A3.

---

## 5. Bond Option Pricing in Short-Rate Models

One of the key advantages of Gaussian short-rate models is closed-form pricing for European options on zero-coupon bonds. This extends to caps, floors, and swaptions through decomposition arguments.

### 5.1 Zero-Coupon Bond Options (ZBO)

For Vasicek, Ho-Lee, and Hull-White one-factor models, Jamshidian (1989) derived the following formula. The price at time $t=0$ of a European call with maturity $T$ on a zero-coupon bond with principal $L$ maturing at time $S \gt T$ is:

$$\boxed{\text{ZBC}(0, T, S, K) = L \cdot P(0,S) \\, \Phi(h) - K \cdot P(0,T) \\, \Phi(h - \sigma_p)}$$

where $K$ is the strike price, $\Phi(\cdot)$ is the standard normal CDF, and:

$$h = \frac{1}{\sigma_p} \ln\frac{L \cdot P(0,S)}{K \cdot P(0,T)} + \frac{\sigma_p}{2}$$

The price of a European put is:

$$\boxed{\text{ZBP}(0, T, S, K) = K \cdot P(0,T) \\, \Phi(-h + \sigma_p) - L \cdot P(0,S) \\, \Phi(-h)}$$

**Volatility Parameter $\sigma_p$**

For Vasicek and Hull-White:

$$\boxed{\sigma_p = \frac{\sigma}{a}\left[1 - e^{-a(S-T)}\right] \sqrt{\frac{1 - e^{-2aT}}{2a}}}$$

For Ho-Lee ($a \to 0$):

$$\sigma_p = \sigma(S-T)\sqrt{T}$$

**Interpretation:** The formula resembles Black's model for bond options, with the forward bond price volatility equal to $\sigma_p/\sqrt{T}$. The term $B(T,S) = \frac{1-e^{-a(S-T)}}{a}$ captures how bond price sensitivity to the short rate depends on the time-to-maturity of the underlying bond.

### 5.2 Jamshidian Decomposition for Coupon Bond Options

In a one-factor short-rate model, all zero-coupon bond prices move in the same direction when $r$ changes: they all increase when $r$ decreases and vice versa. This monotonicity enables a powerful decomposition for options on coupon-bearing bonds.

**The Key Insight:** A European option on a coupon-bearing bond can be expressed as a portfolio of European options on the constituent zero-coupon bonds.

**Procedure (Jamshidian, 1989):**

**Step 1:** Find the critical short rate $r^{\star}$ at option maturity $T$ for which the coupon bond price equals the strike $K$:

$$\sum_{i=1}^{n} c_i \\, P(T, T_i; r^{\star}) = K$$

where $c_i$ is the $i$-th cash flow at time $T_i \gt T$ (coupons plus principal), and $P(T, T_i; r^{\star})$ is the bond price when $r(T) = r^{\star}$.

In affine models, this becomes:

$$\sum_{i=1}^{n} c_i \\, A(T, T_i) \\, e^{-B(T, T_i) \\, r^{\star}} = K$$

This is a nonlinear equation in $r^{\star}$ that must be solved numerically (e.g., Newton-Raphson).

**Step 2:** Compute the component strikes. For each payment date $T_i$:

$$X_i = A(T, T_i) \\, e^{-B(T, T_i) \\, r^{\star}}$$

**Step 3:** The coupon bond option price is the sum of ZCB option prices:

$$\boxed{\text{CBO}(0, T, \\{T_i\\}, \\{c_i\\}, K) = \sum_{i=1}^{n} c_i \cdot \text{ZBO}(0, T, T_i, X_i)}$$

where ZBO is a call for a call option on the bond, and a put for a put option.

**Why This Works:** At maturity $T$:
- If $r(T) \lt r^{\star}$: all constituent ZCBs are worth more than their strikes $X_i$, so the call on each ZCB is in-the-money, and the coupon bond exceeds $K$
- If $r(T) \gt r^{\star}$: all ZCBs are worth less than their strikes, all options expire worthless, and the coupon bond is below $K$

The decomposition holds because the exercise decision is the same for all components.

### 5.3 Caps and Floors as ZCB Option Portfolios

A caplet paying at time $T_i$ based on the rate set at $T_{i-1}$ can be rewritten as a put option on a zero-coupon bond. Specifically, a caplet with strike rate $K$, notional $N$, and accrual period $\tau_i$ has payoff at $T_i$:

$$N \cdot \tau_i \cdot \max(L(T_{i-1}, T_i) - K, 0)$$

This is equivalent to $(1 + K\tau_i)$ put options on a ZCB with face value $\frac{N}{1+K\tau_i}$, maturing at $T_i$, struck at $\frac{1}{1+K\tau_i}$, with the option expiring at $T_{i-1}$.

**Cap Pricing Formula (Vasicek/Hull-White):**

$$\boxed{\text{Cap}(0, \mathcal{T}, N, K) = N \sum_{i=1}^{n} \left[ P(0, T_{i-1}) \\, \Phi(-h_i + \sigma_p^i) - (1 + K\tau_i) \\, P(0, T_i) \\, \Phi(-h_i) \right]}$$

where $\mathcal{T} = \\{T_0, T_1, \ldots, T_n\\}$ is the payment schedule and:

$$\sigma_p^i = \frac{\sigma}{a}\left[1 - e^{-a(T_i - T_{i-1})}\right] \sqrt{\frac{1 - e^{-2aT_{i-1}}}{2a}}$$

$$h_i = \frac{1}{\sigma_p^i} \ln\frac{P(0, T_i)(1 + K\tau_i)}{P(0, T_{i-1})} + \frac{\sigma_p^i}{2}$$

**Floor Pricing:**

$$\text{Floor}(0, \mathcal{T}, N, K) = N \sum_{i=1}^{n} \left[ (1 + K\tau_i) \\, P(0, T_i) \\, \Phi(h_i) - P(0, T_{i-1}) \\, \Phi(h_i - \sigma_p^i) \right]$$

### 5.4 European Swaptions via Jamshidian

A payer swaption is economically equivalent to a call option on a coupon-bearing bond. Consider a swaption giving the right to enter a payer swap at time $T$ with payment dates $\\{T_1, \ldots, T_n\\}$ and fixed rate $K$:

- Define cash flows: $c_i = K\tau_i$ for $i = 1, \ldots, n-1$ and $c_n = 1 + K\tau_n$
- Apply Jamshidian decomposition with these cash flows
- The strike on the coupon bond option is $K = 1$ (par)

**Payer Swaption Price:**

$$\boxed{\text{PS}(0, T, \mathcal{T}, N, K) = N \sum_{i=1}^{n} c_i \cdot \text{ZBP}(0, T, T_i, X_i)}$$

**Receiver Swaption Price:**

$$\text{RS}(0, T, \mathcal{T}, N, K) = N \sum_{i=1}^{n} c_i \cdot \text{ZBC}(0, T, T_i, X_i)$$

### 5.5 CIR Bond Option Pricing

For the CIR model, bond option prices involve the non-central chi-squared distribution. The transition density of $r(T)$ given $r(t)$ follows a scaled non-central chi-squared distribution, which makes the option pricing formula more complex than the Gaussian case.

Hull notes that CIR bond options "involve integrals of the noncentral chi-square distribution." The explicit formula (Cox, Ingersoll, Ross 1985) is:

$$\text{ZBC}_{\text{CIR}}(0, T, S, K) = P(0,S) \cdot \chi^2(2r^{\star}[\rho + \psi + B(T,S)]; \\, \nu, \\, \lambda_1) - K \cdot P(0,T) \cdot \chi^2(2r^{\star}[\rho + \psi]; \\, \nu, \\, \lambda_2)$$

where $\chi^2(x; \nu, \lambda)$ is the CDF of the non-central chi-squared distribution with $\nu$ degrees of freedom and non-centrality parameter $\lambda$. The parameters involve:

$$\nu = \frac{4ab}{\sigma^2}, \quad \rho = \frac{2a}{\sigma^2(e^{aT} - 1)}, \quad \psi = \frac{a + \gamma}{\sigma^2}$$

$$r^{\star} = \frac{\ln(A(T,S)/K)}{B(T,S)}$$

and $\lambda_1$, $\lambda_2$ depend on the current short rate $r(0)$.

The non-central chi-squared CDF can be computed via series expansions or numerical integration. This makes CIR option pricing computationally heavier than Vasicek/Hull-White, though still tractable.

---

## 6. Hull-White Trinomial Tree Construction

For pricing American options, Bermudan swaptions, and other path-dependent or early-exercise structures, analytical formulas are unavailable. Trees provide a flexible numerical framework that exactly matches the initial term structure.

### 6.1 The Two-Stage Procedure

Hull and White (1994) developed a robust two-stage procedure for building trinomial trees:

**Stage 1:** Build a tree for a "base" process $R^{\star}$ that is mean-reverting around zero:
$$dR^{\star} = -a R^{\star} \\, dt + \sigma \\, dW$$

**Stage 2:** Shift the tree nodes by time-dependent amounts $\alpha_i$ to match the observed term structure, giving the tree for $R = R^{\star} + \alpha$.

### 6.2 Stage 1: Building the $R^{\star}$ Tree

**Node Spacing:** Set the vertical spacing between rates as:

$$\boxed{\Delta R = \sigma \sqrt{3 \Delta t}}$$

This choice minimizes discretization error.

**Node Labeling:** Let $(i, j)$ denote the node at time $t = i\Delta t$ with rate $R^{\star} = j\Delta R$.

**Branching Patterns:** Three patterns are used:

| Pattern | Description | When Used |
|---------|-------------|-----------|
| Standard | up/middle/down | Most nodes |
| Up-biased | up2/up1/middle | Low rate region ($j \lt j_{\min}$) |
| Down-biased | middle/down1/down2 | High rate region ($j \gt j_{\max}$) |

The thresholds are set to ensure positive probabilities:

$$j_{\max} = \lceil 0.184 / (a \Delta t) \rceil, \quad j_{\min} = -j_{\max}$$

**Branching Probabilities:** For standard branching at node $(i, j)$:

$$\boxed{p_u = \frac{1}{6} + \frac{1}{2}(a^2 j^2 \Delta t^2 - aj\Delta t)}$$

$$\boxed{p_m = \frac{2}{3} - a^2 j^2 \Delta t^2}$$

$$\boxed{p_d = \frac{1}{6} + \frac{1}{2}(a^2 j^2 \Delta t^2 + aj\Delta t)}$$

These match the conditional mean $\mathbb{E}[\Delta R^{\star}] = -aR^{\star}\Delta t$ and variance $\text{Var}[\Delta R^{\star}] = \sigma^2\Delta t$.

**Derivation of Probabilities (Brigo-Mercurio Framework)**

The probabilities above are a special case of the general trinomial tree matching conditions. Brigo and Mercurio (Section 3.3.3, eq. 3.50) derive probabilities by requiring that the discrete tree matches the continuous-time mean and variance of the OU process.

For a general trinomial tree with node spacing $\Delta x$ where the process can move from node $j$ to nodes $k-1$, $k$, or $k+1$, the probabilities must satisfy:

$$p_u + p_m + p_d = 1 \quad \text{(probabilities sum to 1)}$$
$$p_u(x_{k+1} - x_j) + p_m(x_k - x_j) + p_d(x_{k-1} - x_j) = \mathbb{E}[\Delta x] \quad \text{(match mean)}$$
$$p_u(x_{k+1} - x_j)^2 + p_m(x_k - x_j)^2 + p_d(x_{k-1} - x_j)^2 = \text{Var}[\Delta x] + (\mathbb{E}[\Delta x])^2 \quad \text{(match variance)}$$

For the Hull-White base process $R^{\star}$ with $\mathbb{E}[\Delta R^{\star}] = -aR^{\star}\Delta t = -aj\Delta R \cdot \Delta t$ and $\text{Var}[\Delta R^{\star}] = \sigma^2\Delta t$, solving this system with $\Delta R = \sigma\sqrt{3\Delta t}$ yields the formulas above.

The key insight is that the choice $\Delta R = \sigma\sqrt{3\Delta t}$ simplifies the algebra: with this spacing, the variance condition becomes $\frac{1}{3}(p_u + p_d) = \frac{1}{3}$, which allows the mean-matching constraint to determine the asymmetry between $p_u$ and $p_d$.

### 6.3 Stage 2: Fitting the Term Structure

**Arrow-Debreu Prices:** Define $Q_{i,j}$ as the present value of a security paying \$1 if node $(i,j)$ is reached and zero otherwise.

**Iterative Procedure:**

1. Initialize: $Q_{0,0} = 1$, $\alpha_0 = -\ln(P^M(0, \Delta t))/\Delta t$ (matches first discount factor)

2. For each time step $i$, compute $\alpha_i$ to match the $(i+1)\Delta t$ discount factor:

$$\boxed{\alpha_i = \frac{1}{\Delta t} \ln\left( \frac{\sum_{j} Q_{i,j} \\, e^{-j\Delta R\\, \Delta t}}{P^M(0, (i+1)\Delta t)} \right)}$$

3. Compute Arrow-Debreu prices for the next time step:

$$Q_{i+1,j} = \sum_k Q_{i,k} \cdot q(k,j) \cdot e^{-(\alpha_i + k\Delta R)\Delta t}$$

where $q(k,j)$ is the transition probability from $(i,k)$ to $(i+1,j)$.

**Final Tree:** Each node $(i,j)$ has short rate $R_{i,j} = \alpha_i + j\Delta R$.

### 6.4 Pricing on the Tree

**Backward Induction:** For a derivative with terminal payoff $V_N(j)$ at time $N\Delta t$:

1. Set terminal values: $V_{N,j} = \text{payoff at node } (N,j)$

2. Roll back: For $i = N-1, \ldots, 0$:
$$V_{i,j} = e^{-R_{i,j}\Delta t} \left[ p_u V_{i+1,j+1} + p_m V_{i+1,j} + p_d V_{i+1,j-1} \right]$$

3. For American/Bermudan options, compare with early exercise value at each step:
$$V_{i,j} = \max\left( \text{continuation value}, \text{exercise value} \right)$$

### 6.5 Worked Example: Trinomial Tree Construction

**Parameters:** $a = 0.1$, $\sigma = 0.01$, $\Delta t = 1$ year

**Step 1:** Compute spacing: $\Delta R = 0.01\sqrt{3} = 0.01732$

**Step 2:** Compute branching threshold: $j_{\max} = \lceil 0.184/0.1 \rceil = 2$

**Step 3:** Build $R^{\star}$ tree probabilities at $j = 0$ (central node):
- $p_u = \frac{1}{6} + 0 = 0.1667$
- $p_m = \frac{2}{3} = 0.6667$
- $p_d = \frac{1}{6} + 0 = 0.1667$

At $j = 1$:
- $p_u = \frac{1}{6} + \frac{1}{2}(0.01 - 0.1) = 0.1217$
- $p_m = \frac{2}{3} - 0.01 = 0.6567$
- $p_d = \frac{1}{6} + \frac{1}{2}(0.01 + 0.1) = 0.2217$

**Step 4:** Given term structure with $R(0,1) = 3.824\\%$, $R(0,2) = 4.512\\%$:
- $\alpha_0 = 0.03824$
- $P^M(0,1) = e^{-0.03824} = 0.9625$
- $P^M(0,2) = e^{-0.04512 \times 2} = 0.9137$

Using forward induction, solve for $\alpha_1 = 0.05205$, giving:
- Node B rate: $0.05205 + 0.01732 = 6.94\\%$
- Node C rate: $0.05205 = 5.21\\%$
- Node D rate: $0.05205 - 0.01732 = 3.47\\%$

### 6.6 Pricing a Bermudan Swaption

**Setup:** 3-year Bermudan payer swaption, exercisable annually, on a 5-year swap.

**Procedure:**

1. Build Hull-White tree with appropriate time steps (at least at exercise dates)

2. At terminal nodes (year 3), compute the value of entering the 2-year tail swap

3. At earlier exercise dates, compare:
   - **Continuation value:** discounted expected value from holding
   - **Exercise value:** value of entering remaining swap

4. Take the maximum and continue rolling back

**Key Insight:** The tree naturally handles the fact that at each exercise date, the remaining swap has different tenor. The Bermudan premium over European comes from optionality at intermediate dates.

---

## 7. "What They Fit and Why" — Model Comparison Table

| Feature | Vasicek | CIR | Hull–White (1F Extended Vasicek) |
|---------|---------|-----|----------------------------------|
| **State variable** | $r_t$ | $r_t$ | $r_t$ |
| **SDE under pricing measure** | $dr = a(b-r)dt + \sigma dW$ | $dr = a(b-r)dt + \sigma\sqrt{r}\\,dW$ | $dr = [\theta(t) - ar]dt + \sigma dW$ |
| **Positivity** | No (Gaussian; can be negative) | Non-negative; strict positivity if $2ab \geq \sigma^2$ | No (Gaussian; can be negative) |
| **Volatility behavior** | Constant $\sigma$ | $\propto \sqrt{r}$ (level-dependent) | Constant $\sigma$ (Gaussian) |
| **Fits $P^M(0,T)$ exactly (as-is)** | Generally no (time-homogeneous limitation) | Generally no (same issue) | Yes, by choosing $\theta(t)$ from initial curve and using $A(t,T)$ built from $P(0,T)$ |
| **Analytical tractability** | ZCB closed form; often analytical bond options | ZCB closed form; analytical in summaries | ZCB affine with explicit $A, B$; used as exogenous term structure Gaussian model |
| **Typical "why used" (supported)** | Simple mean reversion; tractable; simulation and intuition; but not exact curve fit | Positivity + level-dependent vol; tractable; but not exact curve fit | Exact initial curve fit + tractability; used (at least) for risk management and for building analyzable pricing frameworks |

---

### 7.1 Model Selection Decision Tree

Choosing the right short-rate model is a critical decision that balances tractability, calibration quality, and computational cost. This section provides systematic guidance for practitioners.

**7.1.1 The Core Trade-offs**

Every short-rate model makes trade-offs across three dimensions:

| Dimension | What You're Trading Off |
|-----------|------------------------|
| **Tractability** | Closed-form prices vs numerical methods |
| **Realism** | Rate positivity, volatility structure, curve dynamics |
| **Calibration** | Fit to initial curve, caps, swaptions, or all three |

No model excels at all three. The choice depends on your product and constraints.

**7.1.2 Decision Flowchart**

```
START: What product are you pricing?
│
├─► Vanilla caps/floors
│   └─► Do you need closed-form prices for speed?
│       ├─► YES: Hull-White (or Black-Karasinski if positivity required)
│       └─► NO: Any model works; Hull-White simplest
│
├─► European swaptions
│   └─► Is swaption cube fit critical?
│       ├─► YES: Consider LMM (Appendix A4) or two-factor G2++
│       └─► NO: Hull-White via Jamshidian decomposition
│
├─► Bermudan swaptions
│   └─► Hull-White tree is the standard
│       └─► For better cube fit: Two-factor G2++ or BK
│
├─► Exotic (CMS, spread options, etc.)
│   └─► Does payoff depend on curve shape?
│       ├─► YES: Two-factor model required (G2++)
│       └─► NO: Hull-White often sufficient
│
├─► PFE / XVA simulation
│   └─► Is positivity a hard constraint?
│       ├─► YES: CIR++ or Black-Karasinski
│       └─► NO: Hull-White (most common) or G2++ for accuracy
│
└─► General risk management
    └─► Hull-White is the industry standard
```

**7.1.3 Model Selection by Product**

| Product | Recommended Model | Rationale |
|---------|------------------|-----------|
| **ATM caps/floors** | Hull-White | Closed-form, fast calibration |
| **OTM caps/floors** | Hull-White or BK | HW for speed; BK if smile matters |
| **European swaptions (ATM)** | Hull-White | Jamshidian decomposition |
| **European swaptions (full cube)** | G2++ or LMM | One-factor can't fit the cube |
| **Bermudan swaptions** | Hull-White tree | Industry standard; BK or G2++ for better fit |
| **CMS products** | G2++ or LMM | Curve dynamics critical |
| **Callable bonds** | Hull-White tree | Simple, effective |
| **PFE/XVA** | Hull-White | Speed for Monte Carlo; G2++ for accuracy |
| **Emerging market (high rates)** | CIR or CIR++ | Level-dependent volatility |
| **Negative rate environment** | Hull-White | Allows negative rates naturally |
| **Strict positivity required** | CIR++ or BK | Guaranteed positive rates |

**7.1.4 Model Selection by Constraint**

| Constraint | Best Choice | Avoid |
|------------|-------------|-------|
| **Must be fast** | Hull-White | Black-Karasinski |
| **Must fit initial curve exactly** | Hull-White, CIR++, BK | Basic Vasicek, CIR |
| **Must have positive rates** | CIR++, BK | Hull-White, Vasicek |
| **Must fit swaption cube** | G2++, LMM | One-factor models |
| **Must be simple to implement** | Hull-White | Two-factor, BK |
| **Must capture curve dynamics** | G2++ | One-factor models |

**7.1.5 Common Mistakes in Model Selection**

1. **Using Hull-White when positivity matters:** In certain risk systems, negative rates cause failures. Use CIR++ or BK instead.

2. **Using one-factor for spread products:** CMS spread options, curve steepeners, and similar products require two-factor dynamics.

3. **Using Black-Karasinski when speed matters:** BK's lack of closed-form prices makes calibration slow. If you don't need strict positivity, use Hull-White.

4. **Calibrating one-factor to the full swaption cube:** This is mathematically impossible—one-factor models have only 2 degrees of freedom after curve fitting. Choose a subset (co-terminal or diagonal) and accept misfit elsewhere.

5. **Using basic Vasicek/CIR for derivatives:** These cannot fit the initial curve, causing immediate P&L breaks. Always use extended versions (Hull-White, CIR++).

> **Desk Reality: The Pragmatic Choice**
>
> In practice, most rates desks use Hull-White as their default model:
> - **For vanilla products:** Hull-White with cap/floor calibration
> - **For Bermudan swaptions:** Hull-White tree with co-terminal swaption calibration
> - **For XVA:** Hull-White Monte Carlo (sometimes G2++ for important counterparties)
>
> Two-factor models and Black-Karasinski are deployed selectively when their advantages justify the additional complexity. The key is understanding what you're giving up—every model is a compromise.

---

## 8. Calibration and Implementation Map (Practitioner-Facing)

### 8.1 Concrete Workflow (One-Desk, One-Currency Short-Rate Framework)

**Inputs**

- Market discount curve $P^M(0,T)$ (or equivalently `fM(0,t)`)
- Optionally: market prices for caps/floors/swaptions (these can be related to ZCB options and priced in these frameworks)

**Choose Model**

Vasicek, CIR, or Hull–White(1F).

Decision driver: do you require an exact initial curve fit for pricing and risk explainability (Hull emphasizes this need for derivatives valuation).

**Fit Initial Curve**

- **Basic Vasicek/CIR:** no exact curve-fit mechanism; the implied curve is endogenous and may not match $P^M(0,T)$.
- **Practical workaround in the literature:** deterministic-shift extensions $r_t = x_t + \varphi(t)$ (Vasicek++ / CIR++), which can fit the initial curve while preserving a base-model structure.
- **Hull–White:** compute $\theta(t)$ from the initial forward curve and chosen $(a, \sigma)$:
$$\theta(t) = \frac{\partial F(0,t)}{\partial t} + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$$

**Calibrate Remaining Parameters**

Goal: select $(a, \sigma)$ to match liquid option prices/vols. Hull describes the calibration process as minimizing the sum of squared differences between model and market prices.

### 8.2 Hull-White Calibration to Caps/Floors

**Objective Function:**

$$\min_{a, \sigma} \sum_{i=1}^{N} w_i \left( V_i^{\text{model}}(a, \sigma) - V_i^{\text{market}} \right)^2$$

where $V_i$ are cap/floor prices and $w_i$ are weights (often inverse variance or equal weights).

**Calibration Procedure:**

1. **Fix the discount curve:** Use OIS curve as $P^M(0,T)$

2. **Select calibration instruments:** ATM caps at standard tenors (1Y, 2Y, 3Y, 5Y, 7Y, 10Y)

3. **Initial guess:** Start with $a \approx 0.03$–$0.10$ (typical mean-reversion speeds), $\sigma \approx 0.005$–$0.015$ (short-rate volatility)

4. **Optimize:** Use Levenberg-Marquardt or similar nonlinear least-squares algorithm

5. **Verify fit:** Check model vs market prices across the calibration set

**Worked Example: Calibrating to ATM Caps**

Given market cap vols (Black) for 2Y, 5Y, 10Y ATM caps and a flat 4% term structure:

| Maturity | Market Vol (Black) | Model Vol (HW) | Fit Error |
|----------|-------------------|----------------|-----------|
| 2Y | 18.5% | 18.3% | -0.2% |
| 5Y | 16.2% | 16.4% | +0.2% |
| 10Y | 14.8% | 14.7% | -0.1% |

Calibrated parameters: $a = 0.046$, $\sigma = 0.0089$

### 8.3 Calibration to Swaptions

For swaption calibration, the choice depends on the target product:

- **Co-terminal swaptions:** Same end date, different start dates (e.g., 1Yx9Y, 2Yx8Y, 5Yx5Y into 10Y)—use for Bermudan swaption pricing
- **Diagonal swaptions:** Constant total maturity (e.g., 1Yx1Y, 2Yx2Y, 5Yx5Y)
- **Single expiry row:** All swaptions with same expiry, different tenors

**One-factor limitation:** A one-factor Hull-White model cannot simultaneously fit all swaptions in a cube due to the implied perfect correlation across tenors. The typical approach is to choose a "target set" aligned with the product being priced and accept misfit elsewhere.

### 8.4 Time-Dependent Parameters: Proceed with Caution

Hull and White note that making $a(t)$ or $\sigma(t)$ time-dependent can improve calibration fit but introduces risks:

- **Overfitting:** Too many parameters relative to market data
- **Instability:** Small changes in market data cause large parameter swings
- **Non-physical dynamics:** Forward volatility can become negative or exhibit unrealistic behavior

**Recommendation:** Keep $a$ and $\sigma$ constant unless there is strong economic justification. Use the deterministic drift $\theta(t)$ to fit the curve; accept that one-factor models have limited ability to match the full volatility surface.

### 8.5 Calibration Fit Quality and Model Limitations

A critical aspect of model risk management is understanding what fit quality to expect and what limitations are inherent in the model structure.

**8.5.1 What One-Factor Models Cannot Do**

One-factor Hull-White has exactly two free parameters ($a$, $\sigma$) after fitting the initial curve. This means:

$$\boxed{\text{One-factor models cannot simultaneously fit caps AND swaptions}}$$

The swaption market has $N_{\text{expiry}} \times N_{\text{tenor}}$ points (e.g., a 10×10 cube has 100 swaptions). With only 2 parameters, exact fit is impossible.

**Why This Happens:** In a one-factor model, all forward rates are perfectly correlated. The swaption volatility depends on the covariance structure of forward rates, which is fully determined by a single volatility function. Different swaption strikes/tenors require different covariance structures that a single parameter set cannot simultaneously produce.

Brigo and Mercurio note: "Finally, one can try a joint calibration to caps and swaptions data. Results are usually not completely satisfactory. This may be due both to misalignments between the two markets and to the low number of parameters in the model."

**8.5.2 Typical Calibration Strategies**

Given the impossibility of fitting everything, practitioners choose calibration targets strategically:

| Strategy | What You Calibrate To | Best For |
|----------|----------------------|----------|
| **Cap calibration** | ATM cap volatilities across maturities | Cap/floor pricing, rate exposure |
| **Co-terminal swaption** | Swaptions with same end date (e.g., 1Yx9Y, 2Yx8Y, ... for 10Y) | Bermudan swaptions exercisable into that swap |
| **Diagonal swaption** | Same expiry+tenor sum (e.g., 1Yx1Y, 2Yx2Y, 5Yx5Y) | General swaption exposure |
| **Single expiry row** | All tenors for one expiry (e.g., 1Y×1Y, 1Y×2Y, 1Y×5Y, ...) | Options expiring at that date |

**8.5.3 Expected Fit Quality**

Based on Brigo and Mercurio's calibration studies, typical fit quality for Hull-White:

| Calibration Target | Expected Fit | Misfit Elsewhere |
|-------------------|--------------|------------------|
| ATM caps | Very good (< 5% vol error) | Swaptions: 10-30% vol error |
| Co-terminal swaptions | Good (< 10% vol error) | Other swaptions: 15-40% vol error |
| Mixed (caps + some swaptions) | Moderate (compromise) | Systematic biases |

Brigo and Mercurio observe regarding model fitting: "A model fitting quality can be deeply affected by the specific market conditions one is trying to reproduce... However, the 'usual' situation illustrated in Figure 3.16 represents what we have been commonly witnessing in the Euro market in the last years."

**8.5.4 Diagnostic Checks**

After calibration, verify:

1. **Curve repricing:** Model $P(0,T)$ matches market $P^M(0,T)$ to numerical tolerance
2. **Calibration instrument fit:** Check model vs market prices for all calibration instruments
3. **Out-of-sample test:** Price instruments NOT in calibration set; monitor error patterns
4. **Parameter stability:** Small changes in market data should not cause large parameter swings
5. **Implied volatility term structure:** Does the model's implied vol structure look reasonable?

**Warning Signs:**
- Parameters at boundary (e.g., $a \to 0$ or $a \to \infty$)
- Wild variation in fit across similar instruments
- Calibration not converging or multiple local minima

**8.5.5 Model Risk Implications**

The calibration limitations create model risk:

| Risk | Manifestation | Mitigation |
|------|---------------|------------|
| **Interpolation risk** | Prices for non-calibrated strikes/tenors may be wrong | Include more calibration instruments |
| **Extrapolation risk** | Long-dated products beyond calibration range | Use conservative assumptions |
| **Correlation risk** | Perfect correlation assumption fails for curve trades | Use two-factor model |
| **Smile risk** | ATM calibration misses OTM prices | Consider stochastic vol or local vol extension |

> **Desk Reality: Living with Imperfect Calibration**
>
> Every desk accepts some level of misfit. The practical approach:
>
> 1. **Choose calibration targets aligned with your book:** If you trade Bermudans, calibrate to co-terminal swaptions
> 2. **Document the calibration strategy:** Regulators and risk managers need to understand what you're fitting
> 3. **Reserve for model risk:** The difference between model and market for out-of-sample instruments is an indicator of model uncertainty
> 4. **Monitor calibration drift:** Re-calibrate regularly; investigate large parameter changes
> 5. **Know when to upgrade:** If misfit is consistently large, consider a more sophisticated model (G2++, LMM)

**Produce Outputs**

*Pricing outputs:*
- $P(t,T)$, model-implied zero rates/forwards
- (When available) ZCB option prices, caps/floors, swaptions (via decompositions)

*Risk outputs:*
- State sensitivity: $\partial P / \partial r = -BP$ for ZCBs in affine models
- Scenario generation: simulate $r_t$ paths under a chosen measure (real-world for risk scenarios vs $Q$ for pricing)

---

## 9. Math and Derivations (Step-by-Step)

### 9.1 Risk-Neutral Valuation in Short-Rate Models

For payoff $H$ at $T$, price at $t$ is:

$$V(t) = \mathbb{E}^Q\left[\exp\left(-\int_t^T r_u \\, du\right) H \\;\Big|\\; \mathcal{F}_t\right]$$

which implies for a ZCB ($H = 1$):

$$P(t,T) = \mathbb{E}^Q\left[\exp\left(-\int_t^T r_u \\, du\right) \Big| \\; \mathcal{F}_t\right]$$

**Sanity Check:** $P(t,t) = 1$ because the integral is zero and payoff is immediate.

### 9.2 OU Solution (Vasicek) — Step-by-Step

Start from:
$$dr_t = a(b - r_t) \\, dt + \sigma \\, dW_t$$

**Step 1.** Rewrite:
$$dr_t + a r_t \\, dt = ab \\, dt + \sigma \\, dW_t$$

**Step 2.** Multiply by integrating factor $e^{at}$:
$$e^{at} dr_t + a e^{at} r_t \\, dt = ab \\, e^{at} \\, dt + \sigma e^{at} \\, dW_t$$

**Step 3.** Recognize left-hand side:
$$d(e^{at} r_t) = ab \\, e^{at} \\, dt + \sigma e^{at} \\, dW_t$$

**Step 4.** Integrate from 0 to $t$:
$$e^{at} r_t - r_0 = ab \int_0^t e^{au} \\, du + \sigma \int_0^t e^{au} \\, dW_u$$

**Step 5.** Compute the deterministic integral:
$$\int_0^t e^{au} \\, du = \frac{e^{at} - 1}{a}$$

**Step 6.** Solve for $r_t$:
$$\boxed{r_t = r_0 e^{-at} + b(1 - e^{-at}) + \sigma \int_0^t e^{-a(t-u)} \\, dW_u}$$

matching the sourced closed form.

**Mean and Variance**

The stochastic integral has mean 0, so:
$$\mathbb{E}[r_t] = r_0 e^{-at} + b(1 - e^{-at})$$

By Itô isometry:
$$\text{Var}(r_t) = \sigma^2 \int_0^t e^{-2a(t-u)} \\, du = \sigma^2 \frac{1 - e^{-2at}}{2a}$$

consistent with the stated formula.

**Unit Check:** $\text{Var}(r_t)$ has units $(1/\text{year})^2$.

### 9.3 Affine Bond Pricing Derivation: Vasicek (Hull's PDE + Exponential-Affine Guess)

**Step 0 (PDE Under the Model)**

Hull states that, under Vasicek, $P(t,T)$ satisfies:
$$\frac{\partial P}{\partial t} + a(b-r)\frac{\partial P}{\partial r} + \frac{\sigma^2}{2}\frac{\partial^2 P}{\partial r^2} - rP = 0$$

with boundary $P(T,T) = 1$.

**Step 1 (Guess Exponential-Affine Form)**

Assume:

$$P(t,T) = A(t,T) e^{-B(t,T)r}$$

**Step 2 (Differentiate)**

- $\partial P / \partial r = -B \\, P$
- $\partial^2 P / \partial r^2 = B^2 P$
- $\partial P / \partial t = A_t e^{-Br} - AB_t r e^{-Br} = P(A_t/A - B_t r)$

**Step 3 (Substitute Into PDE, Divide by $P$)**

$$\left(\frac{A_t}{A} - B_t r\right) + a(b-r)(-B) + \frac{\sigma^2}{2}(B^2) - r = 0$$

Collect terms in $r$ and constants:

- **Coefficient of $r$:** $-B_t + aB - 1 = 0$
- **Constant term:** $\frac{A_t}{A} - abB + \frac{\sigma^2}{2}B^2 = 0$

Hull presents the resulting ODEs (equivalently):
$$B_t - aB + 1 = 0, \quad A_t - abAB + \frac{\sigma^2}{2}AB^2 = 0$$

**Step 4 (Solve for $B(t,T)$)**

Solve $B_t - aB + 1 = 0$ backward with boundary $B(T,T) = 0$, giving:
$$\boxed{B(t,T) = \frac{1 - e^{-a(T-t)}}{a}}$$

**Step 5 (Solve for $A(t,T)$)**

With $B$ known, solve the $A$ ODE; Hull provides the closed form:
$$\boxed{A(t,T) = \exp\left(\frac{(B(t,T) - T + t)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B(t,T)^2}{4a}\right)}$$

**Sanity Check:** $T = t \Rightarrow B = 0 \Rightarrow A = 1 \Rightarrow P(t,t) = 1$.

### 9.4 Affine Bond Pricing Derivation: CIR (Hull's PDE + Exponential-Affine Guess)

**Step 0 (PDE)**

Hull's PDE for CIR ZCB pricing:
$$\frac{\partial P}{\partial t} + a(b-r)\frac{\partial P}{\partial r} + \frac{\sigma^2 r}{2}\frac{\partial^2 P}{\partial r^2} - rP = 0$$

**Step 1 (Guess)**
$$P(t,T) = A(t,T) e^{-B(t,T)r}$$

**Step 2 (Substitute)**

Using $\partial P / \partial r = -BP$, $\partial^2 P / \partial r^2 = B^2 P$, substitute into PDE and divide by $P$.

Hull states the resulting ODE system:
$$B_t - aB - \frac{1}{2}\sigma^2 B^2 + 1 = 0, \quad A_t - abAB = 0$$

with boundary $P(T,T) = 1$.

**Step 3 (Closed Forms)**

Hull provides:
$$\gamma = \sqrt{a^2 + 2\sigma^2}$$

$B(t,T)$ and $A(t,T)$ as in Section 3.3.

### 9.5 Hull–White Curve-Fit Derivation Map: How theta(t) Links to P^M(0,T)

**Core Point (Supported)**

Hull gives an explicit $\theta(t)$ in terms of the initial instantaneous forward curve $F(0,t)$ and its derivative:

$$\boxed{\theta(t) = \frac{\partial F(0,t)}{\partial t} + a \\, F(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})}$$

**How This Encodes "Fit the Initial Curve" (Mechanics You Can Verify)**

1. From the market curve $P^M(0,t)$, compute $F(0,t)$ via:
$$F(0,t) = -\frac{\partial \ln P^M(0,t)}{\partial t}$$
(Same definition appears as $f^M(0,t)$ in the short-rate chapter setup.)

2. Having fixed $(a, \sigma)$, compute $\theta(t)$ from the sourced formula.

3. Use Hull–White bond price:
$$P(t,T) = A(t,T)\exp(-B(t,T)r(t)), \quad B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$
and:
$$\ln A(t,T) = \ln\frac{P(0,T)}{P(0,t)} + B(t,T)F(0,t) - \frac{\sigma^2}{4a^3}(e^{-aT} - e^{-at})^2(e^{2at} - 1)$$

**Repricing Check at $t = 0$ (Algebraic Verification)**

Plug $t = 0$ into $\ln A$. The last term vanishes because $e^{2a \cdot 0} - 1 = 0$.

Then $A(0,T) = P(0,T)\exp(B(0,T)F(0,0))$.

Hence:
$$P(0,T) = A(0,T)e^{-B(0,T)r(0)} = P(0,T)\exp(B(0,T)(F(0,0) - r(0)))$$

Setting $r(0) = F(0,0)$ gives exact equality.

This is the clean way to see how the initial curve is "inserted" via $A(t,T)$.

**Connection to Deterministic Shifts (General Exogenous Term Structure Idea)**

The short-rate chapter also presents a general deterministic shift construction $r_t = x_t + \varphi(t)$ leading to bond prices that factor into a deterministic shift term times a base-model bond price, which is another way to view "fitting $P^M(0,T)$ by construction."

### 9.6 Connection to the HJM Framework

The Hull-White model can be understood as a special case of the Heath-Jarrow-Morton (HJM) framework, which models the entire forward rate curve rather than just the short rate. This connection provides deeper insight into when short-rate models are appropriate.

**The HJM Framework (Preview)**

In HJM, the instantaneous forward rate $f(t,T)$ evolves under the risk-neutral measure as:

$$df(t,T) = \mu_f(t,T) \\, dt + \sigma_f(t,T) \\, dW(t)$$

where the drift $\mu_f$ is determined by no-arbitrage conditions (the HJM drift restriction—see Appendix A3).

**The Separable Volatility Condition**

A key result (Andersen & Piterbarg, Proposition 10.1.7): if the forward rate volatility has the "separable" form:

$$\boxed{\sigma_f(t,T) = \sigma(t) \cdot g(T)}$$

then the resulting forward rate dynamics are **Markovian in a finite-dimensional state**—meaning they can be represented as a short-rate model.

**Hull-White as Separable HJM**

In the Hull-White model, the forward rate volatility is:

$$\sigma_f(t,T) = \sigma \cdot e^{-a(T-t)}$$

This is separable with $\sigma(t) = \sigma$ (constant) and $g(T) = e^{aT}$ up to normalization. The exponentially decaying structure ensures that:
1. Short-maturity forwards are more volatile than long-maturity forwards
2. The entire curve can be reconstructed from the single state variable $r(t)$

**Why This Matters**

The HJM connection explains:

| Observation | HJM Explanation |
|-------------|-----------------|
| Hull-White has closed-form bond prices | Separable HJM reduces to affine short-rate |
| Perfect correlation across maturities | Single Brownian motion drives all forwards |
| Volatility decays with maturity | Exponential structure of $\sigma_f(t,T) = \sigma e^{-a(T-t)}$ |
| Two-factor models break correlation | Two Brownian motions → non-separable |

**Practical Implication:** If you need non-exponentially-decaying forward volatility (e.g., humped volatility structures observed in the market), you must either:
1. Make volatility time-dependent (with the instability warnings noted in Section 8.4)
2. Use a two-factor model (G2++)
3. Move to a full HJM or market model framework (Appendix A3-A4)

> **Note:** Full treatment of HJM, including the drift restriction derivation and general volatility structures, is provided in Appendix A3.

---

## 10. Worked Examples (At Least 10 Numeric Examples)

### Conventions for Examples

- Rates are in decimals (e.g., 5% $\to$ 0.05)
- Times are in years
- Exponentials are rounded to 4 d.p. as needed

---

### Example 1: Vasicek — Compute E[r_T] and Var(r_T)

**Given:**
- $a = 0.5$, $b = 0.05$, $\sigma = 0.02$, $r_0 = 0.03$, $T = 2$

**From Vasicek mean/variance:**
$$\mathbb{E}[r_T] = r_0 e^{-aT} + b(1 - e^{-aT}), \quad \text{Var}(r_T) = \frac{\sigma^2}{2a}(1 - e^{-2aT})$$

**Step-by-Step:**

1. $e^{-aT} = e^{-1} = 0.3679$

2. **Mean:**
$$\mathbb{E}[r_2] = 0.03(0.3679) + 0.05(1 - 0.3679) = 0.0110 + 0.05(0.6321) = 0.0110 + 0.0316 = 0.0426$$

3. **Variance:** $\sigma^2 = 0.0004$, $2a = 1$, $e^{-2aT} = e^{-2} = 0.1353$
$$\text{Var}(r_2) = \frac{0.0004}{1}(1 - 0.1353) = 0.0004(0.8647) = 0.0003459$$

4. **Standard deviation:** $\sqrt{0.0003459} = 0.0186$

**Result:** $\mathbb{E}[r_2] \approx 0.0426$, $\text{Var}(r_2) \approx 3.459 \times 10^{-4}$, $\text{SD}(r_2) \approx 0.0186$

---

### Example 2: Vasicek — Compute $B(0,T)$, $A(0,T)$, and $P(0,T)$

**Given:**
- $a = 0.3$, $b = 0.05$, $\sigma = 0.01$, $r_0 = 0.04$, $t = 0$, $T = 2$

**Formulas:**
$$B(0,T) = \frac{1 - e^{-aT}}{a}, \quad P(0,T) = A(0,T) e^{-B(0,T)r_0}$$
$$A(0,T) = \exp\left(\frac{(B(0,T) - T)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B(0,T)^2}{4a}\right)$$

**Step-by-Step:**

1. $e^{-aT} = e^{-0.6} = 0.5488$

2. $B = \frac{1 - 0.5488}{0.3} = \frac{0.4512}{0.3} = 1.5039$

3. Compute $a^2 b - \sigma^2/2$:
   - $a^2 = 0.09$
   - $a^2 b = 0.09 \times 0.05 = 0.0045$
   - $\sigma^2/2 = 0.0001/2 = 0.00005$
   - Difference $= 0.00445$

4. Divide by $a^2$: $\frac{0.00445}{0.09} = 0.049444$

5. First exponent term: $(B - T) \times 0.049444 = (1.5039 - 2) \times 0.049444 = (-0.4961) \times 0.049444 = -0.02455$

6. Second exponent term: $\frac{\sigma^2 B^2}{4a} = \frac{0.0001 \times (1.5039)^2}{1.2}$
   - $B^2 = 2.2617$
   - Numerator $= 0.00022617$
   - Divide by 1.2 gives $0.0001885$

7. Total exponent: $\ln A = -0.02455 - 0.0001885 = -0.02474$
   - $A = e^{-0.02474} = 0.9756$

8. Bond price: $P(0,2) = 0.9756 \times \exp(-1.5039 \times 0.04)$
   - $-1.5039 \times 0.04 = -0.06016$
   - $\exp(-0.06016) = 0.9416$
   - $P = 0.9756 \times 0.9416 = 0.9189$

**Result:** $B(0,2) = 1.5039$, $A(0,2) \approx 0.9756$, $P(0,2) \approx 0.9189$

---

### Example 3: CIR — Check Positivity Intuition and Strict-Positivity Condition

**Given:**
- $a = 0.5$, $b = 0.04$, $\sigma = 0.10$, $r_0 = 0.03$

**Hull's Condition:** If $2ab \geq \sigma^2$, $r(t)$ is never zero; otherwise it can touch zero.

**Compute:**
- $2ab = 2(0.5)(0.04) = 0.04$
- $\sigma^2 = 0.01$
- Since $0.04 \geq 0.01$, the condition holds.

**Result:** The model suggests $r(t)$ stays strictly positive (never hits zero) under this parameter set.

---

### Example 4: CIR — Compute a Sample $P(0,T)$ Using $A, B$

**Use parameters designed for simple arithmetic:**
- $a = 0.3$, $b = 0.05$, $\sigma = 0.2828427$ (so $\sigma^2 = 0.08$)
- $r_0 = 0.04$, $T = 2$, $t = 0$

**Formulas:**
$$\gamma = \sqrt{a^2 + 2\sigma^2}$$

**Step-by-Step:**

1. $\gamma = \sqrt{0.09 + 0.16} = \sqrt{0.25} = 0.5$

2. Compute $e^{\gamma T} = e^1 = 2.7183$, so $e^{\gamma T} - 1 = 1.7183$

3. **Denominator:**
$$(\gamma + a)(e^{\gamma T} - 1) + 2\gamma = (0.5 + 0.3)(1.7183) + 1.0 = 0.8(1.7183) + 1.0 = 1.3746 + 1.0 = 2.3746$$

4. **$B(0,2)$:**
$$B = \frac{2(1.7183)}{2.3746} = \frac{3.4366}{2.3746} = 1.4470$$

5. **Exponent for $A$:**
$$\frac{2ab}{\sigma^2} = \frac{2(0.3)(0.05)}{0.08} = \frac{0.03}{0.08} = 0.375$$

6. **$A$ base ratio:**
$$\frac{2\gamma e^{(a+\gamma)T/2}}{(\gamma + a)(e^{\gamma T} - 1) + 2\gamma} = \frac{1.0 \cdot e^{(0.8) \cdot 1}}{2.3746} = \frac{e^{0.8}}{2.3746} = \frac{2.2255}{2.3746} = 0.9361$$

7. Raise to power 0.375:
   - $\ln A = 0.375 \ln(0.9361)$
   - $\ln(0.9361) \approx -0.0661$
   - $\ln A \approx 0.375(-0.0661) = -0.0248$
   - $A \approx e^{-0.0248} = 0.9755$

8. **Bond price:**
$$P(0,2) = A e^{-B r_0} = 0.9755 \cdot e^{-1.4470 \cdot 0.04}$$
   - Exponent $-1.4470 \cdot 0.04 = -0.05788$
   - $e^{-0.05788} = 0.9438$
   - $P = 0.9755 \times 0.9438 = 0.9200$

**Result:** $B(0,2) \approx 1.4470$, $A(0,2) \approx 0.9755$, $P(0,2) \approx 0.9200$

---

### Example 5: Hull–White — Compute theta(t) From a Toy Initial Curve

**Choose an initial discount curve defined by:**
$$P(0,t) = \exp(-0.02t - 0.0025t^2)$$

**Then:**
$$F(0,t) = -\frac{\partial \ln P(0,t)}{\partial t} = 0.02 + 0.005t, \quad \frac{\partial F(0,t)}{\partial t} = 0.005$$

This forward-rate definition matches the market-forward definition structure.

**Pick Hull–White parameters:**
- $a = 0.1$, $\sigma = 0.01$ ($\sigma^2 = 10^{-4}$)

**Hull's $\theta(t)$:**
$$\theta(t) = F_t(0,t) + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$$

Compute $\frac{\sigma^2}{2a} = \frac{0.0001}{0.2} = 0.0005$.

**Now compute $\theta(t)$ at $t = 1, 2, 3, 4$:**

| $t$ | $F$ | $aF$ | $1 - e^{-2at}$ | Last term | $\theta(t)$ |
|-----|-----|------|----------------|-----------|-------------|
| 1 | 0.025 | 0.0025 | $1 - 0.8187 = 0.1813$ | 0.0000907 | $0.005 + 0.0025 + 0.0000907 = 0.0075907$ |
| 2 | 0.03 | 0.0030 | $1 - 0.6703 = 0.3297$ | 0.0001649 | $0.005 + 0.0030 + 0.0001649 = 0.0081649$ |
| 3 | 0.035 | 0.0035 | $1 - 0.5488 = 0.4512$ | 0.0002256 | $0.005 + 0.0035 + 0.0002256 = 0.0087256$ |
| 4 | 0.04 | 0.0040 | $1 - 0.4493 = 0.5507$ | 0.0002753 | $0.005 + 0.0040 + 0.0002753 = 0.0092753$ |

**Result:** $\theta(1) \approx 0.00759$, $\theta(2) \approx 0.00816$, $\theta(3) \approx 0.00873$, $\theta(4) \approx 0.00928$

---

### Example 6: Hull–White — Repricing Check Using the Fitted Curve

Use the same curve and parameters as Example 5.

**Compute a few market discount factors:**

| Maturity | $P(0,T)$ |
|----------|----------|
| 1 | $e^{-0.0225} = 0.9777$ |
| 2 | $e^{-0.05} = 0.9512$ |
| 3 | $e^{-0.0825} = 0.9208$ |

**From Hull's bond price:**
$$P(0,T) = A(0,T) e^{-B(0,T)r(0)}, \quad \ln A(0,T) = \ln P(0,T) + B(0,T)F(0,0)$$

because the last term in $\ln A(t,T)$ vanishes at $t = 0$.

Take $r(0) = F(0,0) = 0.02$ (since $F(0,t) = 0.02 + 0.005t$).

Then algebraically:
$$P(0,T) = P(0,T)\exp(B(0,T)(F(0,0) - r(0))) = P(0,T)$$

So repricing holds exactly in this idealized setup.

**Numeric Spot-Check (Showing Cancellation Explicitly)**

Pick $T = 2$:

1. $B(0,2) = \frac{1 - e^{-0.2}}{0.1} = \frac{1 - 0.8187}{0.1} = 1.8127$

2. $A(0,2) = P(0,2)\exp(B(0,2)F(0,0)) = 0.9512 \times e^{1.8127(0.02)}$
   - $e^{1.8127(0.02)} = e^{0.036254} = 1.0369$
   - $A(0,2) = 0.9512 \times 1.0369 = 0.9863$

3. Now multiply by $e^{-Br(0)} = e^{-1.8127(0.02)} = e^{-0.036254} = 0.9644$

4. $P(0,2) = 0.9863 \times 0.9644 = 0.9512$ (matches input)

**Result:** The fitted construction reproduces the input discount factors (within rounding).

---

### Example 7: Compare $P(0,T)$ Sensitivity to $r_0$: Vasicek vs CIR

Use Example 2 (Vasicek) and Example 4 (CIR), both with $r_0 = 0.04$, $T = 2$.

**Sensitivity Identity:**
$$\frac{\partial P}{\partial r_0} = -B(0,T) \\, P(0,T)$$

**Vasicek:** $B = 1.5039$, $P = 0.9189$
$$\frac{\partial P}{\partial r_0} = -(1.5039)(0.9189) = -1.3818$$

A +1 bp shift ($\Delta r_0 = 0.0001$) gives $\Delta P \approx -0.0001382$.

**CIR:** $B = 1.4470$, $P = 0.9200$
$$\frac{\partial P}{\partial r_0} = -(1.4470)(0.9200) = -1.3312$$

A +1 bp shift gives $\Delta P \approx -0.0001331$.

**Result:** Under these toy parameters, CIR produces slightly smaller ZCB sensitivity to $r_0$ than Vasicek (because $B$ and $P$ differ).

---

### Example 8: Negative Rates Illustration Under Vasicek/HW — Compute P(r_T < 0)

Because Vasicek is Gaussian with known mean/variance, $r_T \sim N(m, s^2)$.

**Pick parameters so that $m = 0.01$ and $s = 0.01$ at $T = 2$:**

Choose $a = 0.5$, $T = 2$, and set $r_0 = b = 0.01$ so:
$$m = r_0 e^{-aT} + b(1 - e^{-aT}) = 0.01$$

For the variance:
$$s^2 = \frac{\sigma^2}{2a}(1 - e^{-2aT}) = \sigma^2(1 - e^{-2})$$

since $2a = 1$. With $1 - e^{-2} = 0.8647$, choose $\sigma$ so that $s = 0.01$:
$$0.0001 = 0.8647 \\, \sigma^2 \quad \Rightarrow \quad \sigma^2 = 0.0001156 \quad \Rightarrow \quad \sigma = 0.01075$$

**Then:**
$$\mathbb{P}(r_T \lt 0) = \mathbb{P}\left(\frac{r_T - m}{s} \lt \frac{0 - m}{s}\right) = \Phi\left(-\frac{m}{s}\right) = \Phi(-1) = 0.1587$$

**Result:** $\mathbb{P}(r_2 \lt 0) \approx 15.87\\%$ for this toy Gaussian setup.

**Hull–White Note:** Hull–White is also Gaussian (normal short-rate distribution), so the same type of calculation applies once the mean is determined by $\theta(\cdot)$.

---

### Example 9: Mean-Reversion Speed Effect — Vary $a$ and Show Impact on $B$ and Bond Sensitivity

Use Vasicek ZCB sensitivity $\partial P / \partial r = -BP$.

**Hold:** $b = 0.05$, $\sigma = 0.01$, $r_0 = 0.04$, $T = 5$, $t = 0$. Compare:

#### Case A: $a = 0.1$

1. $B = \frac{1 - e^{-0.5}}{0.1} = \frac{1 - 0.6065}{0.1} = 3.9347$

2. Compute $A$ exponent:
   - $a^2 = 0.01$, $a^2 b = 0.0005$, $\sigma^2/2 = 0.00005$, difference $= 0.00045$
   - $(a^2 b - \sigma^2/2)/a^2 = 0.00045/0.01 = 0.045$
   - $B - T = 3.9347 - 5 = -1.0653$, so first term $= -1.0653 \times 0.045 = -0.04794$
   - Second term $\sigma^2 B^2/(4a) = 0.0001(15.480)/(0.4) = 0.00387$
   - $\ln A = -0.04794 - 0.00387 = -0.05181 \Rightarrow A = 0.9495$

3. $P = A e^{-Br_0} = 0.9495 \times e^{-3.9347(0.04)}$
   - Exponent $-0.15739$, $e^{-0.15739} = 0.8544$
   - $P = 0.9495 \times 0.8544 = 0.8110$

4. **Sensitivity:** $\frac{\partial P}{\partial r_0} = -BP = -(3.9347)(0.8110) = -3.190$

#### Case B: $a = 0.5$

1. $B = \frac{1 - e^{-2.5}}{0.5} = \frac{1 - 0.0821}{0.5} = 1.8358$

2. Compute $A$ exponent:
   - $a^2 = 0.25$, $a^2 b = 0.0125$, $\sigma^2/2 = 0.00005$, difference $= 0.01245$
   - $(a^2 b - \sigma^2/2)/a^2 = 0.01245/0.25 = 0.0498$
   - $B - T = 1.8358 - 5 = -3.1642$, first term $= -3.1642 \times 0.0498 = -0.1575$
   - Second term $\sigma^2 B^2/(4a) = 0.0001(3.368)/(2.0) = 0.000168$
   - $\ln A = -0.1575 - 0.000168 = -0.1577 \Rightarrow A = 0.8541$

3. $P = 0.8541 \times e^{-1.8358(0.04)}$
   - Exponent $-0.07343$, $e^{-0.07343} = 0.9292$
   - $P = 0.8541 \times 0.9292 = 0.7932$

4. **Sensitivity:** $\frac{\partial P}{\partial r_0} = -(1.8358)(0.7932) = -1.456$

**Result:** Higher mean reversion $a$ reduces $B$ and reduces the bond's sensitivity to the short rate.

---

### Example 10: Sanity Check — sigma -> 0 Makes the Evolution Deterministic

Use Vasicek with $a = 0.3$, $b = 0.05$, $r_0 = 0.04$, $T = 2$.

**From the exact solution:**
$$r_T = r_0 e^{-aT} + b(1 - e^{-aT}) + \sigma \int_0^T e^{-a(T-u)} \\, dW_u$$

If $\sigma = 0$, the stochastic integral is zero and $r_T$ is deterministic.

**Compute deterministic $r_2$:**

1. $e^{-aT} = e^{-0.6} = 0.5488$

2. $r_2 = 0.04(0.5488) + 0.05(1 - 0.5488) = 0.02195 + 0.05(0.4512) = 0.02195 + 0.02256 = 0.04451$

**Now compare with $\sigma = 0.01$:**

Variance (from formula) is:
$$\text{Var}(r_2) = \frac{0.0001}{2(0.3)}(1 - e^{-1.2}) = \frac{0.0001}{0.6}(1 - 0.3010) = 0.0001667(0.6990) = 0.0001165$$

So SD $\approx 0.01079$.

**Result:**
- $\sigma = 0 \Rightarrow r_2 = 0.04451$ exactly
- $\sigma = 0.01 \Rightarrow r_2$ is random with SD about $1.08\\%$

---

### Example 11: Ho-Lee — Bond Pricing with Time-Dependent Drift

The Ho-Lee model is Hull-White with $a = 0$:
$$dr = \lambda(t) \\, dt + \sigma \\, dW$$

For Ho-Lee, bond pricing simplifies because $B(t,T) = T - t$.

**Given:**
- Market discount curve: $P^M(0,T) = e^{-0.03T - 0.002T^2}$ (upward-sloping curve)
- $\sigma = 0.01$, $r_0 = 0.03$, price 2-year ZCB

**Step-by-Step:**

1. **Market forward rate:** $f^M(0,t) = -\frac{\partial}{\partial t}\ln P^M(0,t) = 0.03 + 0.004t$

2. **Ho-Lee $\lambda(t)$:** From the drift condition (Section 3.7):
$$\lambda(t) = f^M_t(0,t) + \sigma^2 t = 0.004 + 0.0001t$$

3. **$B(0,2) = 2 - 0 = 2$** (trivial for Ho-Lee)

4. **Compute $A(0,2)$:** Using the Ho-Lee bond price formula:
$$\ln A(0,T) = \ln P^M(0,T) + B(0,T)f^M(0,0) - \frac{\sigma^2 T^3}{6}$$

   - $\ln P^M(0,2) = -0.03(2) - 0.002(4) = -0.068$
   - $B(0,2) \cdot f^M(0,0) = 2 \times 0.03 = 0.06$
   - $\frac{\sigma^2 T^3}{6} = \frac{0.0001 \times 8}{6} = 0.000133$
   - $\ln A(0,2) = -0.068 + 0.06 - 0.000133 = -0.008133$
   - $A(0,2) = e^{-0.008133} = 0.9919$

5. **Bond price:** $P(0,2) = A(0,2) \cdot e^{-B(0,2) \cdot r_0}$
   - $e^{-2 \times 0.03} = e^{-0.06} = 0.9418$
   - $P(0,2) = 0.9919 \times 0.9418 = 0.9342$

**Verification:** Check against market curve:
- $P^M(0,2) = e^{-0.068} = 0.9343$ ✓

**Result:** Ho-Lee prices the 2-year ZCB at $P(0,2) = 0.9342$, matching the input curve (within rounding).

---

### Example 12: CIR++ — Checking Positivity of the Shifted Rate

**Setup:** We want to fit a market curve while ensuring rates stay positive.

**Given:**
- CIR base parameters: $k = 0.3$, $\theta = 0.04$, $\sigma = 0.10$, $x_0 = 0.03$
- Market instantaneous forward: $f^M(0,t) = 0.025 + 0.005t$ (upward-sloping)
- Check positivity of $r(t) = x(t) + \varphi(t)$ at $t = 2$

**Step-by-Step:**

1. **Verify Feller condition for base CIR:**
   - $2k\theta = 2(0.3)(0.04) = 0.024$
   - $\sigma^2 = 0.01$
   - Since $0.024 \gt 0.01$, Feller holds $\Rightarrow$ $x(t) \gt 0$ always

2. **Compute $h = \sqrt{k^2 + 2\sigma^2}$:**
   - $h = \sqrt{0.09 + 0.02} = \sqrt{0.11} = 0.3317$

3. **Compute CIR forward rate $f^{CIR}(0,2)$ using Brigo-Mercurio eq. 3.77:**

   Let $e^{2h} = e^{0.6633} = 1.9412$, so $e^{th} - 1 = 0.9412$ at $t = 2$.

   Denominator: $2h + (k+h)(e^{th} - 1) = 0.6633 + (0.6317)(0.9412) = 0.6633 + 0.5946 = 1.2579$

   First term: $\frac{2k\theta(e^{th} - 1)}{1.2579} = \frac{0.024 \times 0.9412}{1.2579} = \frac{0.02259}{1.2579} = 0.01796$

   Second term: $x_0 \frac{4h^2 e^{th}}{[2h + (k+h)(e^{th}-1)]^2}$
   - $4h^2 = 4(0.11) = 0.44$
   - $e^{th} = 1.9412$
   - Numerator: $0.03 \times 0.44 \times 1.9412 = 0.02562$
   - Denominator: $(1.2579)^2 = 1.5823$
   - Second term: $0.02562/1.5823 = 0.01619$

   $f^{CIR}(0,2) = 0.01796 + 0.01619 = 0.03415$

4. **Compute shift function:**
$$\varphi(2) = f^M(0,2) - f^{CIR}(0,2) = (0.025 + 0.01) - 0.03415 = 0.035 - 0.03415 = 0.00085$$

5. **Positivity check:**
   - $x(t) \gt 0$ (guaranteed by Feller)
   - $\varphi(2) = 0.00085 \gt 0$
   - Therefore $r(2) = x(2) + \varphi(2) \gt 0$ ✓

**Result:** With these parameters, $\varphi(t) \gt 0$ at $t = 2$, so positivity is guaranteed. If the market curve were lower (more negative $\varphi$), we would need to verify that $\varphi(t) \gt -\min(x(t))$ across the path distribution.

**Warning:** In a deeply inverted or negative-rate environment, $\varphi(t)$ can become sufficiently negative that $r(t) = x(t) + \varphi(t)$ goes negative despite $x(t) \gt 0$. The CIR++ model does not automatically guarantee positive $r(t)$—only positive $x(t)$.

---

## 11. Practical Notes

### 11.1 Common Pitfalls

**Mixing Measures Without Stating It**

Hull explicitly distinguishes real-world and risk-neutral short-rate processes using market price of risk adjustments. Treating an estimated real-world drift as if it were risk-neutral (or vice versa) can break pricing consistency.

**Confusing "Fit Initial Curve" with "Fit Vol Surface/Term Structure"**

- Deterministic drift/shift (`theta(t)` or `varphi(t)`) fits the yield curve at time 0.
- Allowing time-dependent parameters can be used to also fit volatility term structures, but is described as potentially dangerous due to instability and ad-hoc parameterizations.

**Using Vasicek/HW Without Acknowledging Negative-Rate Implications**

Gaussian models imply normal distributions and can generate negative rates, which may be unacceptable depending on product and horizon.

**Over-Interpreting One-Factor Dynamics**

One-factor models imply perfect correlation between rates of different maturities; this limitation is explicitly highlighted for concrete pricing problems.

### 11.2 Verification Tests (Minimum Viable Model QA)

**Hull–White Curve Repricing**

Ensure that model $P(0,T)$ reproduces the input $P^M(0,T)$ to numerical tolerance (Example 6 shows the algebraic identity in the analytic setup).

**Bond Price Monotonicity in $r_t$**

Verify $\partial P / \partial r = -BP \lt 0$. This is a direct identity in affine Vasicek/CIR.

**Parameter Perturbation Stability**

Small changes in $a, \sigma$ should not lead to discontinuous shifts in key outputs (unless your calibration objective is ill-conditioned).

If you allow time-dependent parameters, re-check stability carefully (the sources warn about instability/overfitting concerns).

---

## 12. Summary & Recall

### 12.1 Executive Summary (13 Bullets)

1. Short-rate models specify an SDE for the instantaneous short rate $r_t$, and price ZCBs via $P(t,T) = \mathbb{E}^Q[\exp(-\int_t^T r_u \\, du) \mid \mathcal{F}_t]$.

2. The bank account numeraire is $B(t) = \exp(\int_0^t r_u \\, du)$.

3. An affine term structure means $P(t,T) = A(t,T) e^{-B(t,T)r_t}$ (equivalently spot rates affine in $r_t$).

4. Vasicek is Gaussian OU: $dr = a(b-r)dt + \sigma dW$, with explicit mean/variance and normal distribution.

5. Vasicek allows negative rates because it is Gaussian.

6. CIR: $dr = a(b-r)dt + \sigma\sqrt{r} \\, dW$; volatility rises with rate level and rates are non-negative.

7. CIR strict positivity is linked to $2ab \geq \sigma^2$ (Feller-type); transition density follows scaled non-central chi-squared with $\nu = 4ab/\sigma^2$ degrees of freedom.

8. Basic time-homogeneous Vasicek/CIR generally do not match the observed initial curve; limited parameters can prevent satisfactory curve calibration.

9. Hull–White extends Vasicek with time-dependent drift $\theta(t)$ so the model can fit the initial discount curve exactly while keeping tractable affine bond prices.

10. Bond options in Gaussian models have closed-form prices; the ZBO formula resembles Black's model with volatility $\sigma_p$ depending on mean-reversion and option/bond maturities.

11. Jamshidian decomposition allows European swaptions to be priced as portfolios of ZCB options by finding the critical short rate $r^{\star}$ where all component options expire together.

12. Hull-White trinomial trees (two-stage construction) enable pricing of American/Bermudan structures while exactly matching the initial term structure.

13. One-factor Gaussian exogenous term structure models are analytically convenient but limited: negative rates and perfect correlation across maturities; still used for risk management.

### 12.2 Cheat Sheet (SDEs + Key Pricing Identities + HW Curve-Fit Step)

**Risk-Neutral Pricing**
$$P(t,T) = \mathbb{E}^Q\left[\exp\left(-\int_t^T r_u \\, du\right) \Big| \mathcal{F}_t\right]$$

**Vasicek**
$$dr = a(b-r)dt + \sigma dW, \quad P(t,T) = A(t,T)e^{-B(t,T)r_t}, \quad B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$
$$A(t,T) = \exp\left(\frac{(B - T + t)(a^2 b - \sigma^2/2)}{a^2} - \frac{\sigma^2 B^2}{4a}\right)$$

**CIR**
$$dr = a(b-r)dt + \sigma\sqrt{r} \\, dW, \quad P(t,T) = A(t,T)e^{-B(t,T)r_t}, \quad \gamma = \sqrt{a^2 + 2\sigma^2}$$
$$B(t,T) = \frac{2(e^{\gamma(T-t)} - 1)}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}, \quad A(t,T) = \left[\frac{2\gamma e^{(a+\gamma)(T-t)/2}}{(\gamma + a)(e^{\gamma(T-t)} - 1) + 2\gamma}\right]^{2ab/\sigma^2}$$

**Sensitivity (Vasicek and CIR)**
$$\frac{\partial P(t,T)}{\partial r(t)} = -B(t,T) P(t,T)$$

**Hull–White 1F**
$$dr = [\theta(t) - ar]dt + \sigma dW, \quad P(t,T) = A(t,T)e^{-B(t,T)r(t)}, \quad B(t,T) = \frac{1 - e^{-a(T-t)}}{a}$$
$$\ln A(t,T) = \ln\frac{P(0,T)}{P(0,t)} + B(t,T)F(0,t) - \frac{\sigma^2}{4a^3}(e^{-aT} - e^{-at})^2(e^{2at} - 1)$$

**Curve-Fit Step**
$$\theta(t) = F_t(0,t) + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$$

**ZCB Option (Vasicek/Hull-White)**
$$\text{ZBC}(0,T,S,K) = L \cdot P(0,S)\Phi(h) - K \cdot P(0,T)\Phi(h - \sigma_p)$$
$$\sigma_p = \frac{\sigma}{a}[1 - e^{-a(S-T)}]\sqrt{\frac{1-e^{-2aT}}{2a}}, \quad h = \frac{1}{\sigma_p}\ln\frac{L \cdot P(0,S)}{K \cdot P(0,T)} + \frac{\sigma_p}{2}$$

**Jamshidian Decomposition**
Find $r^{\star}$ solving $\sum_i c_i P(T,T_i; r^{\star}) = K$, set $X_i = P(T,T_i; r^{\star})$, then $\text{CBO} = \sum_i c_i \cdot \text{ZBO}(0,T,T_i,X_i)$

**Hull-White Tree (Key Formulas)**
$$\Delta R = \sigma\sqrt{3\Delta t}, \quad j_{\max} = \lceil 0.184/(a\Delta t) \rceil$$
$$\alpha_i = \frac{1}{\Delta t}\ln\left(\frac{\sum_j Q_{i,j}e^{-j\Delta R\Delta t}}{P^M(0,(i+1)\Delta t)}\right)$$

### 12.3 Flashcards (50 Prompts with Answers)

| # | Prompt | Answer |
|---|--------|--------|
| 1 | Define the money-market account $B(t)$. | $B(t) = \exp(\int_0^t r_u \\, du)$ |
| 2 | Define the ZCB price $P(t,T)$ in short-rate modeling. | Price at $t$ of payoff 1 at $T$; $P(t,T) = \mathbb{E}^Q[\exp(-\int_t^T r_u \\, du) \mid \mathcal{F}_t]$ |
| 3 | What does "affine term structure" mean (in $r_t$)? | $P(t,T) = A(t,T) e^{-B(t,T)r_t}$ (equivalently spot rate affine in $r_t$) |
| 4 | Vasicek SDE. | $dr = a(b-r)dt + \sigma dW$ |
| 5 | Vasicek mean of $r_t$. | $\mathbb{E}[r_t] = r_0 e^{-at} + b(1 - e^{-at})$ |
| 6 | Vasicek variance of $r_t$. | $\text{Var}(r_t) = \frac{\sigma^2}{2a}(1 - e^{-2at})$ |
| 7 | Why can Vasicek generate negative rates? | $r_t$ is normally distributed (Gaussian) |
| 8 | Vasicek bond pricing form. | $P(t,T) = A(t,T) e^{-B(t,T)r_t}$ |
| 9 | Vasicek $B(t,T)$. | $B(t,T) = (1 - e^{-a(T-t)})/a$ |
| 10 | CIR SDE. | $dr = a(b-r)dt + \sigma\sqrt{r} \\, dW$ |
| 11 | CIR positivity idea. | CIR does not allow negative rates; diffusion term vanishes at 0 |
| 12 | CIR strict-positivity condition in Hull. | If $2ab \geq \sigma^2$, $r(t)$ is never zero |
| 13 | CIR "level-dependent volatility" meaning. | Short-rate volatility scales like $\sqrt{r}$ |
| 14 | CIR bond pricing form. | $P(t,T) = A(t,T) e^{-B(t,T)r(t)}$ with $\gamma = \sqrt{a^2 + 2\sigma^2}$ |
| 15 | Common ZCB sensitivity identity in Vasicek/CIR. | $\partial P / \partial r = -BP$ |
| 16 | What is the market instantaneous forward rate $f^M(0,t)$? | $f^M(0,t) = -\partial_t \ln P^M(0,t)$ |
| 17 | Hull–White 1F SDE. | $dr = [\theta(t) - ar]dt + \sigma dW$ |
| 18 | Hull–White as time-dependent Vasicek. | $dr = a(b(t) - r)dt + \sigma dW$ with $b(t) = \theta(t)/a$ |
| 19 | Hull–White curve-fit formula for $\theta(t)$. | $\theta(t) = F_t(0,t) + aF(0,t) + \frac{\sigma^2}{2a}(1 - e^{-2at})$ |
| 20 | Hull–White bond pricing form. | $P(t,T) = A(t,T) e^{-B(t,T)r(t)}$ |
| 21 | Hull–White $B(t,T)$. | $B(t,T) = (1 - e^{-a(T-t)})/a$ |
| 22 | Why Hull–White can fit the initial curve. | $A(t,T)$ explicitly uses $P(0,T)/P(0,t)$ and $\theta(t)$ is built from the initial forward curve |
| 23 | Why time-homogeneous models struggle to fit $P^M(0,T)$. | They generate a restricted functional family for $P(0,T)$ with few parameters; may not match observed term structure |
| 24 | What is Jamshidian's decomposition used for in this context? | To price European swaptions explicitly by decomposing into bond options (in the short-rate framework) |
| 25 | What is a key limitation of one-factor short-rate models for pricing realism? | One-factor implies perfect correlation across maturities; Gaussian versions also allow negative rates; used more in risk management |
| 26 | Zero-coupon bond call formula in Vasicek/Hull-White. | $\text{ZBC}(0,T,S,K) = L \cdot P(0,S)\Phi(h) - K \cdot P(0,T)\Phi(h-\sigma_p)$ |
| 27 | What is $\sigma_p$ in the ZBO formula (Vasicek/HW)? | $\sigma_p = \frac{\sigma}{a}[1-e^{-a(S-T)}]\sqrt{\frac{1-e^{-2aT}}{2a}}$ |
| 28 | Jamshidian decomposition: what is the critical rate $r^{\star}$? | The short rate at option expiry where coupon bond price equals strike: $\sum_i c_i P(T,T_i; r^{\star}) = K$ |
| 29 | How does Jamshidian decomposition work? | Find $r^{\star}$, compute strikes $X_i = P(T,T_i; r^{\star})$, price coupon bond option as sum of ZCB options |
| 30 | How is a caplet related to a bond option? | A caplet is equivalent to $(1+K\tau)$ put options on a ZCB maturing at the payment date |
| 31 | Hull-White trinomial tree: what is the node spacing $\Delta R$? | $\Delta R = \sigma\sqrt{3\Delta t}$ |
| 32 | What are the two stages of Hull-White tree construction? | Stage 1: Build $R^{\star}$ tree mean-reverting around zero; Stage 2: Shift by $\alpha_i$ to match term structure |
| 33 | What is an Arrow-Debreu price $Q_{i,j}$? | PV of security paying \$1 if node $(i,j)$ is reached, zero otherwise |
| 34 | How is $\alpha_i$ computed in Hull-White tree fitting? | $\alpha_i = \frac{1}{\Delta t}\ln\left(\frac{\sum_j Q_{i,j}e^{-j\Delta R \Delta t}}{P^M(0,(i+1)\Delta t)}\right)$ |
| 35 | CIR transition distribution. | Scaled non-central chi-squared with $\nu = 4ab/\sigma^2$ degrees of freedom |
| 36 | What ensures positive probabilities in Hull-White tree? | Threshold $j_{\max} = \lceil 0.184/(a\Delta t) \rceil$; switch to biased branching beyond thresholds |
| 37 | Standard branching probability $p_m$ in Hull-White tree. | $p_m = \frac{2}{3} - a^2 j^2 \Delta t^2$ |
| 38 | Why can Jamshidian decomposition be applied in one-factor models? | All ZCB prices move in the same direction when $r$ changes; exercise decision is simultaneous for all components |
| 39 | Bermudan swaption pricing on a tree: key comparison at each exercise date. | Compare continuation value (discounted expected value) vs exercise value (value of entering remaining swap) |
| 40 | Calibration objective for Hull-White to caps. | $\min_{a,\sigma} \sum_i w_i (V_i^{\text{model}} - V_i^{\text{market}})^2$ |
| 41 | Ho-Lee SDE. | $dr = \lambda(t) \\, dt + \sigma \\, dW$ (no mean reversion) |
| 42 | How is Ho-Lee related to Hull-White? | Ho-Lee is Hull-White with $a = 0$ |
| 43 | Ho-Lee $B(t,T)$. | $B(t,T) = T - t$ (linear in time-to-maturity) |
| 44 | Why does Ho-Lee produce parallel yield curve shifts? | With $B = T - t$, rate sensitivity is proportional to maturity; all rates shift equally for $\Delta r$ |
| 45 | CIR++ model construction. | $r(t) = x(t) + \varphi(t)$ where $x(t)$ follows basic CIR and $\varphi(t)$ is deterministic shift |
| 46 | What is the shift function $\varphi(t)$ in CIR++? | $\varphi(t) = f^M(0,t) - f^{CIR}(0,t;\alpha)$ (market forward minus model forward) |
| 47 | Why use CIR++ instead of basic CIR? | CIR++ fits the initial curve exactly while preserving CIR's positive-rate property |
| 48 | When might CIR++ produce negative rates? | If $\varphi(t)$ is sufficiently negative (deeply inverted/negative market curve), $r(t) = x(t) + \varphi(t)$ can go negative despite $x(t) \gt 0$ |
| 49 | Vasicek real-world vs risk-neutral drift. | Real-world: $dr = [k\theta - (k+\lambda\sigma)r]dt + \sigma dW^0$; Risk-neutral: $dr = a(b-r)dt + \sigma dW^Q$ |
| 50 | Why distinguish physical and risk-neutral parameters? | Physical (estimated from data) used for simulation/risk; risk-neutral (implied from prices) used for pricing |
| 51 | Black-Karasinski SDE. | $d\ln r(t) = [\theta(t) - a\ln r(t)]dt + \sigma dW(t)$ |
| 52 | Why is Black-Karasinski always positive? | $\ln r$ is Gaussian, so $r = e^{\ln r} \gt 0$ always (exponential of any real number is positive) |
| 53 | Key limitation of Black-Karasinski. | No closed-form bond prices; must use tree or PDE for all pricing |
| 54 | Trade-off: Hull-White vs Black-Karasinski. | HW: analytically tractable but allows negative rates; BK: guarantees positivity but requires numerical methods |
| 55 | G2++ model short rate formula. | $r(t) = x(t) + y(t) + \varphi(t)$ where $x$, $y$ are correlated OU processes |
| 56 | Why use two-factor models? | One-factor implies perfect correlation across maturities; two-factor allows imperfect correlation for spread products |
| 57 | Why does Euler discretization fail for CIR? | Near zero, Euler can produce negative values, breaking the $\sqrt{r}$ diffusion term |
| 58 | What is the QE scheme for? | Accurate CIR simulation; combines quadratic and exponential sampling to handle near-zero rates efficiently |
| 59 | Can one-factor models fit the full swaption cube? | No; one-factor has only 2 free parameters after curve fitting, insufficient for the full cube |
| 60 | What is "co-terminal" swaption calibration? | Calibrating to swaptions with same end date (e.g., 1Yx9Y, 2Yx8Y, ... all ending at year 10); used for Bermudan pricing |
| 61 | Connection between Hull-White and HJM. | Hull-White is separable HJM with $\sigma_f(t,T) = \sigma e^{-a(T-t)}$; exponential decay makes dynamics Markovian in short rate |
| 62 | When to use Hull-White vs CIR++. | HW: when negative rates acceptable, speed important; CIR++: when positivity required, partial tractability acceptable |

---

## 13. Mini Problem Set (24 Questions)

*Brief solution sketches for questions 1–8 only.*

---

**1. Derive the OU solution for Vasicek using an integrating factor.**

*Sketch:* Multiply by $e^{at}$, integrate, solve for $r_t$. Match the closed form in the notes.

---

**2. Using the Vasicek solution, compute $\mathbb{E}[r_t]$ and $\text{Var}(r_t)$.**

*Sketch:* Take expectation (stochastic integral mean 0) and apply Itô isometry for variance to obtain $\sigma^2(1 - e^{-2at})/(2a)$.

---

**3. Starting from Hull's Vasicek PDE, show that the exponential-affine ansatz implies ODEs for $A$ and $B$.**

*Sketch:* Substitute $P = Ae^{-Br}$ into the PDE, compute derivatives, collect $r$ and constant terms to recover Hull's ODEs.

---

**4. Solve the Vasicek B-ODE and recover the closed form for B(t,T).**

Given ODE: $B_t - aB + 1 = 0$ with $B(T,T) = 0$.

*Sketch:* Solve linear ODE; $B(t,T) = (1 - e^{-a(T-t)})/a$.

---

**5. In CIR, explain why volatility is "level dependent" and write the diffusion coefficient.**

*Sketch:* Diffusion term is $\sigma\sqrt{r}$; hence standard deviation in small time $\propto \sqrt{r}$.

---

**6. Using Hull's CIR positivity statement, test whether the given parameters satisfy strict positivity.**

Given: $a = 0.2$, $b = 0.03$, $\sigma = 0.15$.

*Sketch:* Compute $2ab = 0.012$, $\sigma^2 = 0.0225$; condition fails, so it may touch zero.

---

**7. Show $\partial P / \partial r = -BP$ given $P = Ae^{-Br}$.**

*Sketch:* Differentiate: $\partial_r P = -BAe^{-Br} = -BP$.

---

**8. In Hull–White, explain how $P(0,T)$ is recovered from the bond pricing formula when $r(0) = F(0,0)$.**

*Sketch:* At $t = 0$, the last term in $\ln A$ is zero; $A(0,T) = P(0,T)\exp(BF(0,0))$; multiply by $\exp(-Br(0))$ to recover $P(0,T)$.

---

**9. Using the deterministic shift idea $r_t = x_t + \varphi(t)$, describe how bond prices factor into a deterministic term and a base-model term.**

---

**10. Compare the shape flexibility of Vasicek/CIR term structures as functions of $r(t)$ and parameters.**

---

**11. For Vasicek, compute the probability $r_T \lt 0$ given $r_T \sim N(m, s^2)$.**

---

**12. For CIR, write gamma and interpret its role in A(t,T) and B(t,T).**

---

**13. Explain why one-factor models imply perfect correlation between rates of different maturities.**

---

**14. Discuss why fitting time-dependent $a(t)$, $\sigma(t)$ might improve calibration but be "dangerous" in practice.**

---

**15. Outline how caps can be priced as portfolios of ZCB options in the short-rate framework.**

---

**16. Outline how European swaptions can be priced via Jamshidian decomposition in a one-factor short-rate model.**

---

**17. For Hull-White, compute sigma_p for a ZCB option.**

Given: $a = 0.05$, $\sigma = 0.01$, $T = 2$, $S = 5$.

*Sketch:* $B(T,S) = \frac{1-e^{-0.05 \times 3}}{0.05} = 2.77$; variance term $= \frac{1-e^{-0.2}}{0.1} = 1.81$; $\sigma_p = 0.01 \times 2.77 \times \sqrt{1.81} \approx 0.0373$.

---

**18. In Hull-White tree construction, compute Delta R and j_max.**

Given: $a = 0.1$, $\sigma = 0.01$, $\Delta t = 0.5$.

*Sketch:* $\Delta R = 0.01\sqrt{1.5} = 0.01225$; $j_{\max} = \lceil 0.184/0.05 \rceil = 4$.

---

**19. Compute branching probabilities at node j = 2 in the R-star tree.**

Given: $a = 0.1$, $\Delta t = 0.5$.

*Sketch:*
- $a^2 j^2 \Delta t^2 = 0.01 \times 4 \times 0.25 = 0.01$
- $aj\Delta t = 0.1$
- $p_u = \frac{1}{6} + \frac{1}{2}(0.01 - 0.1) = 0.122$
- $p_m = \frac{2}{3} - 0.01 = 0.657$
- $p_d = \frac{1}{6} + \frac{1}{2}(0.01 + 0.1) = 0.222$

---

**20. For CIR, compute the transition degrees of freedom nu and verify the Feller condition.**

Given: $a = 0.3$, $b = 0.05$, $\sigma = 0.1$.

*Sketch:* $\nu = 4ab/\sigma^2 = 4 \times 0.3 \times 0.05/0.01 = 6$. Check: $2ab = 0.03 \geq \sigma^2 = 0.01$ ✓ (strict positivity).

---

**21. (Model Selection)**

A desk needs to price a 5-year Bermudan swaption exercisable annually into a 10-year swap. They also need to compute PFE for XVA. Recommend models for each use case and justify.

*Sketch:*
- **Bermudan pricing:** Hull-White trinomial tree, calibrated to co-terminal swaptions (1Yx14Y, 2Yx13Y, ... 5Yx10Y all ending at year 15). One-factor sufficient for single Bermudan; two-factor (G2++) if better cube fit needed.
- **PFE:** Hull-White Monte Carlo for speed (thousands of paths needed). If counterparty is important and curve dynamics matter, consider G2++.

---

**22. (CIR Simulation)**

Given CIR parameters: `a = 0.5`, `b = 0.02`, `sigma = 0.15`, `r0 = 0.01`.
**(a)** Verify the Feller condition.
**(b)** What simulation method should you use and why?
**(c)** If you use Euler with reflection, what bias do you expect?

*Sketch:*
- (a) Compute `2ab = 0.02` and `sigma^2 = 0.0225`. Since `0.02 < 0.0225`, Feller is violated (`nu < 2`).
- (b) Use QE scheme or exact simulation. Euler with reflection will introduce significant bias since the process can touch zero.
- (c) Reflection artificially increases the process near zero, biasing paths upward. The bias is most severe when starting near zero (as here, `r0 = 0.01`).

---

**23. (Calibration Limitations) Explain why a one-factor Hull-White model cannot simultaneously fit ATM cap volatilities and the full swaption volatility surface. What is the mathematical reason?**

*Sketch:* After fitting the initial curve with `theta(t)`, Hull-White has only two free parameters (`a` and `sigma`). The swaption surface has many more degrees of freedom (for example, 100 points in a 10x10 cube). Mathematically, a one-factor model implies perfect correlation across all forward-rate maturities, but different swaption expiry/tenor combinations require richer correlation structures. Under perfect correlation, model-implied swaption volatilities are pinned down by `a` and `sigma`, so the model cannot reproduce the full observed market surface.

---

**24. (Black-Karasinski) Show lognormality of r(t) under OU dynamics for ln r(t).**

Assume:

$$
d\ln r = [\theta(t) - a\ln r]dt + \sigma dW
$$

Question: given `r(s)`, what is the conditional mean of `ln r(t)`?

*Sketch:* The OU process for `ln r` has explicit solution:

$$
\ln r(t) = \ln r(s) e^{-a(t-s)} + \int_s^t e^{-a(t-u)}\theta(u)du + \sigma\int_s^t e^{-a(t-u)}dW(u)
$$

The stochastic integral is Gaussian (Itô integral of a deterministic function), so `ln r(t)` conditional on `F_s` is Gaussian. Therefore `r(t) = exp(ln r(t))` is lognormal.

Conditional mean:

$$
\mathbb{E}[\ln r(t) \mid r(s)] = \ln r(s)e^{-a(t-s)} + \int_s^t e^{-a(t-u)}\theta(u)du
$$

---

## References

- John C. Hull, *Options, Futures, and Other Derivatives* (Vasicek/CIR/Hull–White; bond options; trinomial trees; curve fitting)
- Damiano Brigo & Fabio Mercurio, *Interest Rate Models: Theory and Practice* (short-rate model families; calibration; Jamshidian decomposition; tree/MC implementation)
- Leif B. G. Andersen & Vladimir V. Piterbarg, *Interest Rate Modeling* (curve-fitting frameworks; shifted models; simulation schemes; practical modeling trade-offs)
- Paul Glasserman, *Monte Carlo Methods in Financial Engineering* (simulation accuracy/stability; exact and approximate sampling for common diffusions)
- Bruce Tuckman & Angel Serrat, *Fixed Income Securities* (rates-model intuition and practical framing)

---

*Appendix A2 of Fixed Income: Practice and Theory*
