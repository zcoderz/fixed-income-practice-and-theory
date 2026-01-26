# Appendix A4: Market Models — LMM (BGM) and Swap Market Model; Calibration Logic

---

## Introduction

Black formulas work. Cap dealers quote caplet volatilities; swaption desks quote swaption volatilities. The market speaks Black. What if your interest rate model *guaranteed* that these quotes translated directly into model prices—no approximations, no residual basis, no awkward calibration gymnastics?

This is the central promise of **market models**. Rather than specifying some abstract short rate $r_t$ and hoping the resulting forward and swap rate distributions happen to look vaguely lognormal, market models *start* with forward rates or swap rates as the modeled quantities and *engineer* their dynamics so that the relevant options are exactly Black-compatible under an appropriate measure. The LIBOR Market Model (LMM, also called BGM after Brace-Gatarek-Musiela) models discrete forward rates; the Swap Market Model (LSM, for Lognormal Swap Model) models swap rates directly.

The price of this convenience is complexity elsewhere. When you simulate all forwards jointly under a common measure, you discover that no-arbitrage imposes **coupled drifts**—each forward's drift depends on the current levels of other forwards. Calibration becomes a high-dimensional optimization balancing caplet fits, swaption fits, correlation structure, and numerical stability. And the original LIBOR-based formulation must adapt to the post-crisis world of SOFR and other risk-free rates.

This appendix provides the complete framework:

- **Section A4.1** establishes conventions and notation for tenor grids, discount factors, and measures
- **Section A4.2** develops the LMM: forward rate dynamics, the crucial coupled-drift structure, and caplet pricing
- **Section A4.3** covers the Swap Market Model: swap rate dynamics, coterminal swaptions, and the LMM-LSM relationship
- **Section A4.4** connects market models to the HJM framework and explains drift restrictions
- **Section A4.5** details calibration methodology: Rebonato's swaption volatility approximation, terminal correlation, correlation parameterizations, objective functions, regularization, and global vs local approaches
- **Section A4.6** addresses simulation, pricing, the two-curve framework, and adaptation to SOFR/RFR
- **Section A4.7** provides full mathematical derivations including the Girsanov-based drift coupling proof
- **Section A4.8** presents 13 worked examples with complete numeric detail

By the end, you will understand why market models became the industry standard for interest rate exotics, how to calibrate them to liquid instruments, and what practical challenges remain.

---

## A4.1 Conventions and Notation

### A4.1.1 Tenor Grid and Accrual Fractions

We work on a discrete tenor grid $\{T_0 < T_1 < \cdots < T_n\}$ with accrual fractions $\delta_i = \tau(T_i, T_{i+1})$ representing the day-count year-fraction for period $[T_i, T_{i+1}]$. As Andersen and Piterbarg emphasize, these are the standard coupon/reset/payment times for money-market FRAs and swap legs.

A "LIBOR-style" simply-compounded forward rate over $[T_i, T_{i+1}]$ is defined from bond prices via:

$$F_i(t) \equiv F(t; T_i, T_{i+1}) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right)$$

Equivalently:

$$P(t, T_{i+1}) = \frac{P(t, T_i)}{1 + \delta_i F_i(t)}$$

### A4.1.2 Money-Market Account and Discount Factors

With short rate $r_t$, the money-market account is:

$$B(t) = B_0 \exp\left(\int_0^t r_s \, ds\right)$$

The stochastic discount factor over $[t, T]$ can be written as:

$$D(t, T) = \frac{B(t)}{B(T)} = \exp\left(-\int_t^T r_s \, ds\right)$$

### A4.1.3 Measures and Numeraires

**Risk-neutral (money-market) measure $\mathbb{Q}$:** Numeraire $B(t)$. Under this measure, discounted tradable prices are martingales.

**$T$-forward measure $\mathbb{Q}^T$:** Numeraire $P(t, T)$. Pricing formula:

$$\Pi_t = P(t, T) \, \mathbb{E}^T\left[H_T \mid \mathcal{F}_t\right]$$

**Discrete $T_i$-forward measure $\mathbb{Q}^i$:** Shorthand for $\mathbb{Q}^{T_i}$, with numeraire $P(t, T_i)$.

**Swap (annuity) measure $\mathbb{Q}^{\alpha,\beta}$:** Numeraire is the swap annuity:

$$C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$$

Under this measure, the forward swap rate is a martingale.

**Terminal measure $\mathbb{Q}^n$:** Uses $P(t, T_n)$ as numeraire. Convenient for simulation because the terminal bond serves as a universal deflator.

**Spot measure $\mathbb{Q}^d$:** Uses a discretely-compounded bank account:

$$B_d(t) = \frac{P(0, T_{\beta(t)})}{P(t, T_{\beta(t)})} \prod_{j=0}^{\beta(t)-1}(1 + \delta_j F_j(T_j))$$

where $\beta(t)$ is the index of the first tenor date after $t$.

### A4.1.4 Black Building Block

Following Andersen and Piterbarg, we define the Black function for call/put sign $\omega \in \{+1, -1\}$:

$$\boxed{\mathrm{Bl}(K, F, v, \omega) = \omega\left(F \, \Phi(\omega d_1) - K \, \Phi(\omega d_2)\right)}$$

where:

$$d_1 = \frac{\ln(F/K) + \frac{1}{2}v^2}{v}, \qquad d_2 = d_1 - v$$

and $\Phi(\cdot)$ is the standard normal CDF.

### A4.1.5 Notation Glossary

| Category | Symbol | Definition |
|----------|--------|------------|
| **Tenor** | $T_i$ | Tenor dates |
| | $\delta_i$ | Accrual fraction for $[T_i, T_{i+1}]$ |
| **Bonds/Discounting** | $P(t, T)$ | Zero-coupon bond price |
| | $r_t$ | Short rate |
| | $B(t)$ | Money-market account |
| | $D(t, T)$ | Discount factor: $\exp\left(-\int_t^T r_s \, ds\right)$ |
| **Forward rates** | $F_i(t)$ | Forward rate over $[T_i, T_{i+1}]$ |
| **Swap rates** | $C_{\alpha,\beta}(t)$ | Swap annuity |
| | $S_{\alpha,\beta}(t)$ | Forward swap rate |
| **Black function** | $\Phi(\cdot)$ | Standard normal CDF |
| | $\mathrm{Bl}(K, F, v, \omega)$ | Black building block |
| **Volatilities** | $\sigma_i(t)$ | Instantaneous vol of $F_i$ |
| | $\sigma_{\alpha,\beta}$ | Swaption Black implied vol |
| **Correlation** | $\rho_{i,j}$ | Instantaneous correlation between $F_i$ and $F_j$ |

---

## A4.2 LMM (BGM) Essentials: Modeling Forward Rates

### A4.2.1 Modeling Target and Design Philosophy

The LMM models the set of forward rates $\{F_i(t)\}_{i=0}^{n-1}$ defined on a discrete tenor grid. As Brigo and Mercurio explain, the key insight is to specify dynamics for tradable rates (forwards) rather than an abstract short rate, ensuring that caplet pricing aligns directly with market quoting conventions.

**Design choice:** Under the $T_{i+1}$-forward measure $\mathbb{Q}^{i+1}$ (numeraire $P(t, T_{i+1})$), the forward $F_i(t)$ is a martingale. In the lognormal-forward model (LFM), it is postulated to follow driftless lognormal dynamics:

$$\boxed{dF_i(t) = \sigma_i(t) F_i(t) \, dZ_i^{i+1}(t)}$$

This is *not* a theorem about how rates actually behave—it is a modeling choice that guarantees Black caplet pricing.

### A4.2.2 Why Lognormal $F_i$ Implies Black Caplet Pricing

Consider a caplet on period $[T_i, T_{i+1}]$, set at $T_i$ and paid at $T_{i+1}$. The payoff is $\delta_i(F_i(T_i) - K)^+$ paid at $T_{i+1}$.

Using the $T_{i+1}$-forward measure:

$$\mathrm{Cpl}_0 = P(0, T_{i+1}) \, \mathbb{E}^{i+1}\left[\delta_i(F_i(T_i) - K)^+\right]$$

Since $F_i$ is lognormal under $\mathbb{Q}^{i+1}$:

$$\ln F_i(T_i) = \ln F_i(0) - \frac{1}{2}\int_0^{T_i} \sigma_i^2(s) ds + \int_0^{T_i} \sigma_i(s) dZ_i^{i+1}(s)$$

The expectation is exactly a Black call on a lognormal forward:

$$\boxed{\mathrm{Cpl}_0 = P(0, T_{i+1}) \cdot \delta_i \cdot \mathrm{Bl}\left(K, F_i(0), v_i, +1\right)}$$

where $v_i = \sqrt{\int_0^{T_i} \sigma_i^2(s) ds}$ is the integrated volatility (for constant $\sigma$, simply $v_i = \sigma_i \sqrt{T_i}$).

### A4.2.3 Coupled Drift Under Other Measures: Step-by-Step Derivation

This is the heart of LMM complexity. When simulating all forwards under a common measure, no-arbitrage forces state-dependent drifts.

#### Under the Terminal Measure $\mathbb{Q}^n$

Andersen and Piterbarg (Lemma 14.2.2) derive the dynamics of $F_k$ under the terminal measure. The key steps:

**Step 1: Start with the bond ratio representation.**

Since $F_k(t) = \frac{1}{\delta_k}\left(\frac{P(t,T_k)}{P(t,T_{k+1})} - 1\right)$, we can write:

$$1 + \delta_k F_k(t) = \frac{P(t, T_k)}{P(t, T_{k+1})}$$

**Step 2: Apply Itô's lemma to the bond ratio.**

Under any equivalent martingale measure, bond ratios follow diffusions. The drift depends on which numeraire we choose.

**Step 3: Change from $\mathbb{Q}^{k+1}$ to $\mathbb{Q}^n$.**

Under $\mathbb{Q}^{k+1}$, $F_k$ is driftless. To change to $\mathbb{Q}^n$, we apply Girsanov's theorem with Radon-Nikodym derivative:

$$\frac{d\mathbb{Q}^{k+1}}{d\mathbb{Q}^n}\bigg|_{\mathcal{F}_t} = \frac{P(t, T_{k+1})/P(0, T_{k+1})}{P(t, T_n)/P(0, T_n)} = \frac{P(t, T_{k+1}) P(0, T_n)}{P(t, T_n) P(0, T_{k+1})}$$

**Step 4: Express the bond ratio as a product.**

$$\frac{P(t, T_{k+1})}{P(t, T_n)} = \prod_{j=k+1}^{n-1} \frac{P(t, T_j)}{P(t, T_{j+1})} = \prod_{j=k+1}^{n-1} (1 + \delta_j F_j(t))$$

**Step 5: Apply Itô to the product.**

Taking the log differential and using the dynamics of each $F_j$:

$$d\ln\left(\prod_{j=k+1}^{n-1}(1+\delta_j F_j)\right) = \sum_{j=k+1}^{n-1} \frac{\delta_j dF_j}{1+\delta_j F_j} - \frac{1}{2}\sum_{j=k+1}^{n-1}\frac{\delta_j^2 \langle dF_j \rangle}{(1+\delta_j F_j)^2}$$

**Step 6: Extract the drift shift.**

The Girsanov drift shift adds:

$$\mu_k^{(n)}(t) = -\sigma_k(t) F_k(t) \sum_{j=k+1}^{n-1} \frac{\delta_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \delta_j F_j(t)}$$

**Result (Terminal Measure $\mathbb{Q}^n$):**

$$\boxed{dF_k(t) = -\sigma_k(t) F_k(t) \sum_{j=k+1}^{n-1} \frac{\delta_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \delta_j F_j(t)} \, dt + \sigma_k(t) F_k(t) \, dZ_k^n(t)}$$

#### Under the Spot Measure $\mathbb{Q}^d$

Andersen and Piterbarg (Lemma 14.2.3) show that under the spot measure, earlier forwards pick up *positive* drift:

$$\boxed{dF_k(t) = \sigma_k(t) F_k(t) \sum_{j=\beta(t)}^{k} \frac{\delta_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \delta_j F_j(t)} \, dt + \sigma_k(t) F_k(t) \, dZ_k^d(t)}$$

where $\beta(t)$ is the index of the first tenor date after $t$.

**Key observation:** The sign of the drift differs between terminal and spot measures. Under the terminal measure, forwards drift *downward* (negative drift correction). Under the spot measure, forwards drift *upward*.

#### Under an Arbitrary Forward Measure $\mathbb{Q}^i$

For $k < i$:

$$dF_k(t) = -\sigma_k(t) F_k(t) \sum_{j=k+1}^{i} \frac{\delta_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \delta_j F_j(t)} \, dt + \sigma_k(t) F_k(t) \, dZ_k^i(t)$$

For $k > i$:

$$dF_k(t) = \sigma_k(t) F_k(t) \sum_{j=i+1}^{k} \frac{\delta_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \delta_j F_j(t)} \, dt + \sigma_k(t) F_k(t) \, dZ_k^i(t)$$

### A4.2.4 Multifactor Extension

In practice, we model forward rate dynamics with a vector volatility:

$$dF_i(t) = \mu_i(t) dt + F_i(t) \boldsymbol{\sigma}_i(t)^\top d\mathbf{Z}(t)$$

where $\boldsymbol{\sigma}_i(t) \in \mathbb{R}^d$ is a $d$-dimensional volatility vector and $\mathbf{Z}(t)$ is a $d$-dimensional Brownian motion. The instantaneous correlation structure is:

$$\rho_{i,j} = \frac{\boldsymbol{\sigma}_i(t)^\top \boldsymbol{\sigma}_j(t)}{|\boldsymbol{\sigma}_i(t)| \cdot |\boldsymbol{\sigma}_j(t)|}$$

The number of factors $d$ controls the richness of correlation structure that can be captured.

---

## A4.3 Swap Market Model Essentials

### A4.3.1 Swap Rate and Annuity

For a swap spanning $[T_\alpha, T_\beta]$ with payments at $\{T_{\alpha+1}, \ldots, T_\beta\}$:

**Swap annuity (PVBP):**

$$C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$$

**Forward swap rate:**

$$\boxed{S_{\alpha,\beta}(t) = \frac{P(t, T_\alpha) - P(t, T_\beta)}{C_{\alpha,\beta}(t)}}$$

The numerator represents the PV of the floating leg (for a forward-starting par swap), and the denominator is the PV of a unit fixed annuity.

### A4.3.2 Swap Measure and Swaption Pricing

The payer swaption payoff at expiry $T_\alpha$ is:

$$\mathrm{Payoff}_{T_\alpha} = (S_{\alpha,\beta}(T_\alpha) - K)^+ \cdot C_{\alpha,\beta}(T_\alpha)$$

Under the swap (annuity) measure $\mathbb{Q}^{\alpha,\beta}$ with numeraire $C_{\alpha,\beta}(t)$:

1. The annuity qualifies as a numeraire (it's strictly positive and tradable)
2. The forward swap rate $S_{\alpha,\beta}(t)$ is a martingale

If we assume $S_{\alpha,\beta}$ follows lognormal dynamics under $\mathbb{Q}^{\alpha,\beta}$:

$$dS_{\alpha,\beta}(t) = \sigma_{\alpha,\beta}(t) S_{\alpha,\beta}(t) dZ^{\alpha,\beta}(t)$$

Then the swaption price is:

$$\boxed{PS^{\mathrm{Black}}_0 = C_{\alpha,\beta}(0) \cdot \mathrm{Bl}\left(K, S_{\alpha,\beta}(0), \sigma_{\alpha,\beta}\sqrt{T_\alpha}, +1\right)}$$

### A4.3.3 LMM vs Swap Market Model: Natural Fit

| Model | Natural Fit | Challenge |
|-------|-------------|-----------|
| **LMM** | Caplets are exactly Black under each $T_{i+1}$-forward measure | Swap rates are *not* exactly lognormal; swaption pricing requires approximation or Monte Carlo |
| **Swap Market Model** | Swaptions are exactly Black under each swap measure | Forward rates are *not* exactly lognormal; cap pricing requires approximation |

**Incompatibility theorem:** Brigo and Mercurio prove that forward rates cannot all be lognormal while swap rates are also lognormal. The two models are mutually exclusive in the strict sense. However, the practical difference is often small—the swap rate distribution under LMM is typically close to lognormal.

### A4.3.4 Coterminal Swaptions and Bermudan Pricing

**Coterminal swaptions** are swaptions with the same end date but different start dates. For example, 1Y×4Y, 2Y×3Y, 3Y×2Y, 4Y×1Y are all coterminal (ending at year 5).

Andersen and Piterbarg emphasize that **coterminal swaption calibration is critical for Bermudan swaption pricing**. A Bermudan swaption allows exercise at multiple dates, and the optimal exercise decision depends on comparing continuation value against immediate exercise value—which is a coterminal swaption payoff.

**Calibration strategy for Bermudans:**

1. Identify the exercise dates $\{T_{\alpha_1}, T_{\alpha_2}, \ldots\}$
2. For each exercise date, the immediate exercise value involves a coterminal swaption
3. Calibrate the LMM to the corresponding coterminal swaption volatilities
4. This ensures the model correctly prices the boundary conditions

**Coterminal swap rate formula:** For a swap from $T_\alpha$ to $T_n$ (fixed terminal date):

$$S_{\alpha,n}(t) = \frac{P(t, T_\alpha) - P(t, T_n)}{C_{\alpha,n}(t)} = \frac{P(t, T_\alpha) - P(t, T_n)}{\sum_{k=\alpha+1}^{n} \delta_k P(t, T_k)}$$

### A4.3.5 Relationship Between LMM and LSM

The swap rate is a weighted average of forward rates. Specifically:

$$S_{\alpha,\beta}(t) = \sum_{i=\alpha}^{\beta-1} w_i(t) F_i(t)$$

where the weights $w_i(t)$ are time-dependent and depend on the current term structure:

$$w_i(t) = \frac{\delta_i P(t, T_{i+1})}{C_{\alpha,\beta}(t)}$$

**Implication for volatility:** The swap rate volatility is approximately:

$$\sigma_{S}^2 \approx \sum_{i,j} w_i w_j \rho_{i,j} \sigma_i \sigma_j$$

This weighted-covariance formula explains why correlation structure in LMM affects swaption prices and why calibration to swaptions constrains correlation parameters.

---

## A4.4 Connection to HJM and Drift Restrictions

### A4.4.1 HJM Drift Restriction Review

In the HJM framework, instantaneous forward rates $f(t, T)$ satisfy under the risk-neutral measure:

$$df(t, T) = \sigma_f(t, T)^\top \left(\int_t^T \sigma_f(t, u) \, du\right) dt + \sigma_f(t, T)^\top dW(t)$$

Under the $T$-forward measure, the drift vanishes:

$$df(t, T) = \sigma_f(t, T)^\top dW^T(t)$$

**Key insight:** Once you choose the volatility structure $\sigma_f(t, T)$, no-arbitrage completely determines the drift.

### A4.4.2 LMM as Discrete HJM

The LMM can be viewed as a discrete-tenor specialization of HJM:

| HJM (Continuous) | LMM (Discrete) |
|------------------|----------------|
| Continuous maturity $T$ | Discrete tenor $T_i$ |
| Instantaneous forward $f(t, T)$ | Simply-compounded forward $F_i(t)$ |
| Volatility $\sigma_f(t, T)$ | Volatility $\sigma_i(t)$ |
| Drift from HJM condition | Drift from measure change (coupled drift) |

The coupled drift formulas in LMM are the discrete-tenor analog of the HJM drift restriction. Both enforce no-arbitrage given a volatility specification.

### A4.4.3 Design Message

Market models represent a specific philosophy:

1. **Choose which rate should be "nice"** (driftless lognormal)
2. **Select the appropriate measure** for that rate
3. **Accept complexity elsewhere**—drift coupling under other measures

This tradeoff is explicit and intentional, unlike short-rate models where the relationship between $r_t$ dynamics and forward/swap rate distributions is indirect.

---

## A4.5 Calibration Methodology

### A4.5.1 Calibration Instruments

#### Caps/Floors → Forward Rate Volatility

Caps are commonly quoted via Black implied volatilities. Since LMM makes caplets exactly Black under forward measures, calibration proceeds:

1. **Bootstrap caplet volatilities** from cap quotes
2. **Fit instantaneous volatility** $\sigma_i(t)$ so that integrated variance matches implied variance:

   $$\int_0^{T_i} \sigma_i^2(s) ds = (\sigma_i^{\text{impl}})^2 T_i$$

#### Swaptions → Swap Rate Volatility and Correlation

Swaption implied volatilities constrain the correlation structure because swap rates are weighted averages of forwards. Calibrating to swaptions involves:

1. Choosing a correlation parameterization
2. Solving for parameters that minimize swaption pricing errors

### A4.5.2 Volatility Parameterizations

**General Piecewise Constant (GPC):** Define $\sigma_i(t)$ as piecewise constant between tenor dates. This is flexible but can have many parameters.

**Parametric Form (Rebonato-style):**

$$\sigma_i(t) = \Phi_i \left[(a(T_{i-1} - t) + d) e^{-b(T_{i-1} - t)} + c\right]$$

Parameters:
- $a, b, c, d$: shape parameters controlling time-to-maturity effects
- $\Phi_i$: scaling factors that can be pinned to caplet volatilities

This gives a parsimonious representation with interpretable behavior:
- The term $(T_{i-1} - t)$ measures time-to-fixing
- The exponential decay $e^{-b(T_{i-1}-t)}$ captures the "hump" shape often observed in volatility term structures

### A4.5.3 Correlation Parameterizations

**Exponential form:**

$$\rho_{i,j} = \exp(-\beta |T_i - T_j|), \qquad \beta > 0$$

Higher $\beta$ means less correlation between distant forwards.

**Rebonato 3-parameter form:**

$$\rho_{i,j} = \rho_\infty + (1 - \rho_\infty) \exp\left(-\beta |T_i - T_j|^\gamma\right)$$

This allows control over:
- Long-range correlation floor ($\rho_\infty$)
- Decay rate ($\beta$)
- Decay curvature ($\gamma$)

**Reduced-rank formulations:** Express volatility vectors in terms of $d$ factors where $d < n$. This reduces computational burden in simulation and ensures the correlation matrix remains positive semi-definite.

**Eigenvalue zeroing algorithm** (Brigo & Mercurio, Section 6.9):
1. Start with full correlation matrix $\rho$ (size $n \times n$)
2. Compute eigenvalue decomposition: $\rho = V \Lambda V^T$ where $\Lambda = \text{diag}(\lambda_1, \ldots, \lambda_n)$
3. Zero out the smallest $n - d$ eigenvalues: $\tilde{\Lambda} = \text{diag}(\lambda_1, \ldots, \lambda_d, 0, \ldots, 0)$
4. Reconstruct: $\tilde{\rho} = V \tilde{\Lambda} V^T$
5. Rescale diagonal to 1 if needed

This produces a rank-$d$ correlation matrix that is automatically positive semi-definite (since all remaining eigenvalues are positive).

**Rebonato's angle parameterization:** An alternative that guarantees valid correlations by construction. Define factor loadings via angles $\theta_{i,k}$:
$$b_{i,1} = \cos\theta_{i,1}, \quad b_{i,k} = \sin\theta_{i,1}\cdots\sin\theta_{i,k-1}\cos\theta_{i,k}$$

Then $\rho_{i,j} = \sum_{k=1}^{p} b_{i,k} b_{j,k}$, which is automatically a valid correlation entry.

### A4.5.4 Rebonato's Swaption Volatility Approximation

A critical formula for calibration is **Rebonato's approximation** for swaption volatility in terms of forward volatilities and correlations. This bridges the LMM (which models forwards) to swaption prices (which the market quotes).

**Setup:** Consider a swap over periods $[\alpha+1, \beta]$ with forward rates $F_{\alpha+1}, \ldots, F_\beta$ and corresponding volatilities $\sigma_i(t)$. Define swap rate weights:

$$w_i(t) = \frac{\delta_i P(t, T_i)}{C_{\alpha,\beta}(t)}$$

Note that $\sum_{i=\alpha+1}^{\beta} w_i(t) = 1$ (the weights sum to one).

**Rebonato's Approximation (Brigo & Mercurio, Proposition 6.15.1):** The Black implied volatility for a swaption expiring at $T_\alpha$ is approximately:

$$\boxed{\left(v_{\alpha, \beta}^{LMM}\right)^{2} \approx \sum_{i,j=\alpha+1}^{\beta} \frac{w_i(0) w_j(0) F_i(0) F_j(0) \rho_{i,j}}{S_{\alpha,\beta}(0)^2} \int_0^{T_\alpha} \sigma_i(t) \sigma_j(t) dt}$$

where $S_{\alpha,\beta}(0)$ is the initial swap rate.

**Interpretation:**
- The double sum runs over all pairs of forward rates in the swap
- Each pair contributes proportionally to their weights, forward levels, and correlation
- The integral captures the cumulative volatility effect over the option lifetime
- The formula is exact when forward volatilities are deterministic (frozen at initial values)

**Simplified form with constant volatilities:** If $\sigma_i(t) = \sigma_i$ (constant), then:

$$\left(v_{\alpha, \beta}^{LMM}\right)^{2} \approx \sum_{i,j=\alpha+1}^{\beta} \frac{w_i(0) w_j(0) F_i(0) F_j(0) \rho_{i,j} \sigma_i \sigma_j T_\alpha}{S_{\alpha,\beta}(0)^2}$$

**Calibration use:** Given market swaption volatilities, this formula provides an analytical link from LMM parameters to swaption prices. During calibration:
- Forward volatilities $\{\sigma_i\}$ primarily pin caplet prices (exactly, by construction)
- Correlations $\{\rho_{i,j}\}$ are adjusted to match swaption volatilities via this approximation

### A4.5.5 Terminal Correlation

When calibrating to swaptions, it is useful to understand **terminal correlation**—the correlation of forward rates at option expiry, as opposed to their instantaneous correlation.

**Rebonato's terminal correlation approximation:**

$$\text{Corr}_0^{LMM}(F_i(T), F_j(T)) \approx \frac{\int_0^T \sigma_i(t) \sigma_j(t) \rho_{i,j}(t) dt}{\sqrt{\int_0^T \sigma_i(t)^2 dt} \sqrt{\int_0^T \sigma_j(t)^2 dt}}$$

**Exact formula (Brigo & Mercurio, formula 6.71):**

$$\text{Corr}_0^{LMM}(F_i(T), F_j(T)) = \frac{e^{\int_0^T \sigma_i(t) \sigma_j(t) \rho_{i,j}(t) dt} - 1}{\sqrt{(e^{\int_0^T \sigma_i^2(t) dt} - 1)(e^{\int_0^T \sigma_j^2(t) dt} - 1)}}$$

**Intuition:** Even with constant instantaneous correlation $\rho_{i,j}$, the terminal correlation can differ due to:
- Time-varying volatilities $\sigma_i(t)$ (weighted differently over $[0, T]$)
- The nonlinear transformation in the exact formula

For typical parameter values, the approximation is accurate to within a few percent.

### A4.5.6 Calibration Objective Functions

**Basic objective:** Minimize squared pricing errors:

$$\min_{\theta} \sum_{\text{instruments } k} w_k \left(V_k^{\text{model}}(\theta) - V_k^{\text{market}}\right)^2$$

where $\theta$ represents all calibration parameters (volatilities, correlations).

**In implied volatility space:** Often more stable:

$$\min_{\theta} \sum_k w_k \left(\sigma_k^{\text{impl,model}}(\theta) - \sigma_k^{\text{impl,market}}\right)^2$$

### A4.5.7 Regularization and Ringing Control

**The ringing problem:** Andersen and Piterbarg explicitly warn that local/bootstrap-style calibration can produce unstable oscillating ("ringing") volatility surfaces. Small changes in input quotes cause large swings in calibrated parameters.

**Regularization:** Add penalty terms to the objective:

$$\min_{\theta} \left[\sum_k w_k (\text{error}_k)^2 + \lambda \cdot R(\theta)\right]$$

Common regularization terms $R(\theta)$:

1. **Smoothness penalty:** $\sum_i (\sigma_{i+1} - \sigma_i)^2$ penalizes rough volatility surfaces
2. **Tikhonov regularization:** $\|\theta - \theta_0\|^2$ pulls toward prior/reference values
3. **Second-derivative penalty:** $\sum_i (\sigma_{i+1} - 2\sigma_i + \sigma_{i-1})^2$ penalizes curvature

The regularization weight $\lambda$ balances fit quality against stability.

### A4.5.8 Global vs Local Calibration

**Local (row-by-row) calibration:**
- Calibrate to caplet $i$ by fitting $\sigma_i$
- Simple and fast
- Can cause ringing; doesn't ensure consistency across instruments

**Global calibration:**
- Solve for all parameters simultaneously
- Minimizes total objective with regularization
- More stable; better for hedging
- Computationally heavier

Andersen and Piterbarg recommend global calibration with regularization for production use.

### A4.5.9 Joint Caps and Swaptions Calibration

The key insight is that:
- Caplets pin volatility levels directly
- Swaptions pin correlations (since swaption vol depends on forward-forward correlation)

A typical workflow:
1. Fit volatility $\{\sigma_i\}$ to caplets (or cap strip)
2. Fit correlation parameters $\{\rho_{i,j}\}$ to swaption matrix
3. Iterate if necessary, with weights reflecting relative importance

**Weighting choice:** If pricing cap-like products, weight caplets heavily. For swaption-like products (Bermudans), weight swaptions. For a general book, balance based on hedge instrument liquidity.

### A4.5.10 Calibration Workflow Summary

```
1. BUILD INITIAL CURVE
   └─→ Bootstrap P(0,T) from deposits/futures/swaps
   └─→ Compute initial forwards F_i(0)

2. CHOOSE MODEL FAMILY
   └─→ LMM for cap-heavy books
   └─→ Consider coterminal focus for Bermudans

3. CHOOSE PARAMETERIZATION
   └─→ Volatility: GPC or parametric
   └─→ Correlation: exponential, Rebonato, or reduced-rank

4. SET UP OBJECTIVE
   └─→ Include caps, swaptions, or both
   └─→ Add regularization terms
   └─→ Weight instruments by importance

5. OPTIMIZE
   └─→ Use gradient-based or Levenberg-Marquardt
   └─→ Multiple starting points for robustness

6. VALIDATE
   └─→ Reprice calibration instruments
   └─→ Check for ringing
   └─→ Stress test: bump quotes and recalibrate
```

---

## A4.6 Simulation, Pricing, and SOFR Adaptation

### A4.6.1 Simulation Under a Common Measure

For Monte Carlo pricing, choose one measure for all forwards. Common choices:

**Terminal measure $\mathbb{Q}^n$:**
- All forwards have negative drift corrections (except $F_{n-1}$ which is driftless)
- Natural deflator: $P(t, T_n)$

**Spot measure $\mathbb{Q}^d$:**
- Positive drift corrections
- Natural for pricing products with cashflows at multiple dates

### A4.6.2 Discretization Schemes

**Log-Euler scheme:** For forward $F_k$ under terminal measure:

$$\ln F_k(t + \Delta t) = \ln F_k(t) + \mu_k^{(n)}(t)\Delta t - \frac{1}{2}\sigma_k^2(t)\Delta t + \sigma_k(t)\sqrt{\Delta t}\, Z$$

where the drift uses current forward values:

$$\mu_k^{(n)}(t) = -\sigma_k(t) \sum_{j=k+1}^{n-1} \frac{\delta_j F_j(t) \sigma_j(t) \rho_{k,j}}{1 + \delta_j F_j(t)}$$

**Predictor-corrector scheme:** Improves accuracy by evaluating drift at both endpoints:

1. **Predict:** Compute $\tilde{F}_k(t+\Delta t)$ using drift at time $t$
2. **Correct:** Recompute drift using average of $F_k(t)$ and $\tilde{F}_k(t+\Delta t)$
3. **Final step:** Use corrected drift

### A4.6.3 Pricing Various Products

**Caps/Caplets:** Under LMM, caplets are Black by construction. Caps are sums:

$$\text{Cap}_0 = \sum_i \text{Caplet}_i = \sum_i P(0, T_{i+1}) \delta_i \, \mathrm{Bl}(K, F_i(0), v_i, +1)$$

**Swaptions:** Either:
- Use Monte Carlo under a common measure
- Apply analytical approximations (Rebonato's formula approximates swaption vol from forward vols and correlations)

**Bermudans:** Monte Carlo with American option techniques (LSM regression, policy iteration).

**Exotics (CMS, etc.):** Full Monte Carlo simulation, potentially with convexity adjustment analytics for simpler cases.

### A4.6.4 SOFR/RFR Adaptation

The original LMM was built around forward-looking LIBOR rates. Post-transition, rates like SOFR are **backward-looking** and compounded in arrears.

**Key differences:**

| LIBOR-style | SOFR-style |
|-------------|------------|
| Known at period start | Known at period end |
| Simple rate over $[T_i, T_{i+1}]$ | Compounded overnight rates |
| $F_i(T_i)$ is fixing | $F_i(T_{i+1})$ is effective fixing |

**Adaptation approaches:**

1. **Backward-looking forward rate:** Define:
   $$F_i^{\text{SOFR}}(t) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right)$$
   Same formula, but the economic interpretation is the expected compounded overnight rate.

2. **Timing adjustment:** For in-arrears products, the rate is observed at $T_{i+1}$, not $T_i$. This introduces a convexity adjustment:
   $$\mathbb{E}^{i+1}[F_i(T_{i+1})] \neq F_i(0)$$

3. **Simulation timing:** When simulating SOFR-based products, evolve forwards to the *payment* date rather than fixing date.

**Practical note:** Many desks treat SOFR forwards similarly to LIBOR forwards for most purposes, with adjustments for specific features like lookback periods and payment delays. The LMM framework remains applicable with appropriate reinterpretation.

### A4.6.5 Two-Curve Framework

In modern markets, the **discount curve** (often OIS/SOFR-based) is distinct from **projection/index curves** (for specific tenors). Andersen and Piterbarg (Section 15.5) describe two approaches to extending LMM for this:

**Approach 1: HJM Extension**

Model the instantaneous forward rates for both curves:
- Discount curve: $f^D(t, T)$
- Index curve: $f^I(t, T)$

The spread $s(t, T) = f^I(t, T) - f^D(t, T)$ evolves with its own volatility structure, correlated with the discount curve. This maintains the HJM drift restriction for each curve.

**Approach 2: Two-Curve LMM**

Model discrete forward rates for both curves:

$$L_n^I(t) = \frac{1}{\delta_n}\left(\frac{P^I(t, T_n)}{P^I(t, T_{n+1})} - 1\right)$$

$$L_n^D(t) = \frac{1}{\delta_n}\left(\frac{P^D(t, T_n)}{P^D(t, T_{n+1})} - 1\right)$$

Under appropriate measures, each forward is lognormal with its own volatility. The correlation structure now includes:
- Intra-curve correlations: $\text{Corr}(L_i^I, L_j^I)$ and $\text{Corr}(L_i^D, L_j^D)$
- Cross-curve correlations: $\text{Corr}(L_i^I, L_j^D)$

**Pricing with two curves:** For a swap paying index rate $L^I$ and discounted at $P^D$:

$$\text{FloatLeg PV} = \sum_i \delta_i P^D(0, T_{i+1}) \mathbb{E}^{D,i+1}[L_i^I(T_i)]$$

The expectation is under the $T_{i+1}$-forward measure associated with the discount curve, which introduces a convexity adjustment if $L^I$ and $L^D$ are correlated.

**Practical impact:** The two-curve framework is now standard for non-collateralized or foreign-collateralized trades, where basis matters.

---

## A4.7 Mathematical Derivations

### A4.7.1 Forward Rate from Bond Prices

Starting from the no-arbitrage relationship:

$$1 + \delta_i F_i(t) = \frac{P(t, T_i)}{P(t, T_{i+1})}$$

Solving for $F_i(t)$:

$$\boxed{F_i(t) = \frac{1}{\delta_i}\left(\frac{P(t, T_i)}{P(t, T_{i+1})} - 1\right)}$$

**Unit check:** $P(\cdot, \cdot)$ is dimensionless, so $P(t, T_i)/P(t, T_{i+1}) - 1$ is dimensionless. Dividing by $\delta_i$ (years) gives rate units (1/year).

### A4.7.2 Black Caplet Formula Derivation

Under $\mathbb{Q}^{i+1}$, with $F_i$ following:

$$dF_i(t) = \sigma_i F_i(t) dZ^{i+1}(t)$$

The solution is:

$$F_i(T_i) = F_i(0) \exp\left(-\frac{1}{2}\sigma_i^2 T_i + \sigma_i Z^{i+1}(T_i)\right)$$

where $Z^{i+1}(T_i) \sim N(0, T_i)$.

The caplet price:

$$\mathrm{Cpl}_0 = P(0, T_{i+1}) \delta_i \mathbb{E}^{i+1}[(F_i(T_i) - K)^+]$$

Since $\ln F_i(T_i)$ is normal with mean $\ln F_i(0) - \frac{1}{2}\sigma_i^2 T_i$ and variance $\sigma_i^2 T_i$:

$$\mathbb{E}^{i+1}[(F_i(T_i) - K)^+] = F_i(0)\Phi(d_1) - K\Phi(d_2)$$

with:

$$d_1 = \frac{\ln(F_i(0)/K) + \frac{1}{2}\sigma_i^2 T_i}{\sigma_i\sqrt{T_i}}, \qquad d_2 = d_1 - \sigma_i\sqrt{T_i}$$

### A4.7.3 Swap Rate and Annuity Derivation

**Annuity definition:**

$$C_{\alpha,\beta}(t) = \sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$$

**Swap rate from floating leg PV:**

The floating leg PV of a forward-starting swap is:

$$\text{PV}_{\text{float}} = P(t, T_\alpha) - P(t, T_\beta)$$

(This is a standard result: the floating leg of a par swap at inception equals notional minus discounted notional at maturity.)

The fixed leg PV at rate $K$ is:

$$\text{PV}_{\text{fixed}} = K \cdot C_{\alpha,\beta}(t)$$

Setting $\text{PV}_{\text{float}} = \text{PV}_{\text{fixed}}$ and solving for the par rate:

$$\boxed{S_{\alpha,\beta}(t) = \frac{P(t, T_\alpha) - P(t, T_\beta)}{C_{\alpha,\beta}(t)}}$$

**Unit check:** Numerator is dimensionless (bond price difference). Denominator is $\sum \delta_k P \sim$ years. Ratio is 1/year = rate.

### A4.7.4 Drift Coupling Full Derivation (Terminal Measure)

We derive the terminal measure drift formula following Brigo & Mercurio (Proposition 6.3.1). Under the terminal measure $\mathbb{Q}^n$, the numeraire is $P(t, T_n)$.

**Step 1: Starting Point.** Under $\mathbb{Q}^{k+1}$, $F_k$ is a martingale:
$$dF_k = \sigma_k F_k dZ_k^{k+1}$$

**Step 2: Radon-Nikodym Derivative.** To change from $\mathbb{Q}^{k+1}$ to $\mathbb{Q}^n$, we compute:
$$\frac{d\mathbb{Q}^n}{d\mathbb{Q}^{k+1}} = \frac{P(T, T_n)/P(0, T_n)}{P(T, T_{k+1})/P(0, T_{k+1})} = \frac{P(0, T_{k+1})}{P(0, T_n)} \cdot \frac{P(T, T_n)}{P(T, T_{k+1})}$$

**Step 3: Product Formula.** Using the telescoping product:
$$\frac{P(t, T_n)}{P(t, T_{k+1})} = \prod_{j=k+1}^{n-1}\frac{1}{1 + \delta_j F_j(t)}$$

Inverting:
$$\frac{P(t, T_{k+1})}{P(t, T_n)} = \prod_{j=k+1}^{n-1}(1 + \delta_j F_j(t))$$

**Step 4: Girsanov Theorem Application.** Let $L_t = \prod_{j=k+1}^{n-1}(1 + \delta_j F_j(t))$ denote the Radon-Nikodym density process (up to constants). By Itô's lemma:
$$d\ln L_t = \sum_{j=k+1}^{n-1} \frac{\delta_j dF_j}{1+\delta_j F_j} - \frac{1}{2}\sum_{j=k+1}^{n-1}\frac{\delta_j^2 \langle dF_j\rangle}{(1+\delta_j F_j)^2}$$

**Step 5: The Key Sign.** Under Girsanov's theorem, when changing *to* measure $\mathbb{Q}^n$ *from* measure $\mathbb{Q}^{k+1}$, the Brownian motion transforms as:
$$dZ_k^{k+1} = dZ_k^n - d\langle Z_k, \ln L\rangle_t$$

The covariation term is:
$$d\langle Z_k, \ln L\rangle_t = \sum_{j=k+1}^{n-1}\frac{\delta_j F_j \sigma_j \rho_{k,j}}{1+\delta_j F_j}dt$$

Note the **negative sign** in the Girsanov shift: we subtract the covariation when going to the new measure.

**Step 6: Substituting.** Replacing $dZ_k^{k+1}$ in the original SDE:
$$dF_k = \sigma_k F_k dZ_k^{k+1} = \sigma_k F_k \left(dZ_k^n - \sum_{j=k+1}^{n-1}\frac{\delta_j F_j \sigma_j \rho_{k,j}}{1+\delta_j F_j}dt\right)$$

Rearranging:

$$\boxed{dF_k = -\sigma_k F_k \sum_{j=k+1}^{n-1}\frac{\delta_j F_j \sigma_j \rho_{k,j}}{1+\delta_j F_j}dt + \sigma_k F_k dZ_k^n}$$

**Intuition for the Negative Sign:** Under the terminal measure $\mathbb{Q}^n$, the numeraire $P(t, T_n)$ grows faster when later forwards are high. The market prices in this drift, and compensates by making earlier forwards drift downward. Brigo & Mercurio state this as: "The earlier forwards have negative drifts proportional to their covariance with the later forwards."

**Spot Measure (Positive Drift):** For comparison, under the spot measure $\mathbb{Q}^d$, the drift has the opposite sign:
$$dF_k = +\sigma_k F_k \sum_{j=\beta(t)}^{k}\frac{\delta_j F_j \sigma_j \rho_{k,j}}{1+\delta_j F_j}dt + \sigma_k F_k dZ_k^d$$

Here earlier forwards contribute positive drift to later ones—the opposite direction.

---

## A4.8 Worked Examples

### Example 1: Build Tenor Grid and Compute Forward Rates

**Tenor:** $T_0 = 0$, $T_1 = 0.5$, $T_2 = 1.0$, $T_3 = 1.5$

**Accruals:** $\delta_0 = \delta_1 = \delta_2 = 0.5$

**Discount factors:**

| Maturity | $P(0, T)$ |
|----------|-----------|
| 0 | 1.0000 |
| 0.5 | 0.9900 |
| 1.0 | 0.9750 |
| 1.5 | 0.9550 |

**Forward rates:**

$$F_0(0) = \frac{1}{0.5}\left(\frac{1.0000}{0.9900} - 1\right) = 2 \times 0.010101 = 0.020202 \approx 2.02\%$$

$$F_1(0) = \frac{1}{0.5}\left(\frac{0.9900}{0.9750} - 1\right) = 2 \times 0.015385 = 0.030769 \approx 3.08\%$$

$$F_2(0) = \frac{1}{0.5}\left(\frac{0.9750}{0.9550} - 1\right) = 2 \times 0.020942 = 0.041885 \approx 4.19\%$$

### Example 2: ATM Caplet Pricing

**Setup:** Caplet on $[T_1, T_2] = [0.5, 1.0]$, paid at $T_2 = 1.0$

**Inputs:**
- Forward: $F_1(0) = 0.030769$
- Strike: $K = F_1(0)$ (ATM)
- Accrual: $\delta_1 = 0.5$
- Discount factor: $P(0, 1.0) = 0.975$
- Expiry: $T_1 = 0.5$
- Volatility: $\sigma_1 = 28.28\%$ (so $v = \sigma\sqrt{T} = 0.2828 \times 0.7071 = 0.20$)

**Black quantities (ATM, so $\ln(F/K) = 0$):**

$$d_1 = \frac{0 + \frac{1}{2}(0.20)^2}{0.20} = \frac{0.02}{0.20} = 0.10$$

$$d_2 = 0.10 - 0.20 = -0.10$$

$\Phi(0.10) = 0.5398$, $\Phi(-0.10) = 0.4602$

**Black call on forward:**

$$\mathrm{Bl} = F[\Phi(d_1) - \Phi(d_2)] = 0.030769 \times 0.0796 = 0.002449$$

**Caplet price:**

$$\mathrm{Cpl}_0 = P(0, 1.0) \times \delta_1 \times \mathrm{Bl} = 0.975 \times 0.5 \times 0.002449 = 0.001194$$

$$\boxed{\mathrm{Cpl}_0 \approx 0.00119 \text{ per unit notional}}$$

### Example 3: Cap as Sum of Caplets

**Cap on $[0.5, 1.5]$** with strike $K = 3.5\%$ and two caplets.

**Caplet A ($[0.5, 1.0]$):** $F_1 = 3.08\% < K$, $v = 0.20$

$$\ln(F/K) = \ln(0.030769/0.035) = -0.1288$$
$$d_1 = \frac{-0.1288 + 0.02}{0.20} = -0.544, \quad d_2 = -0.744$$
$$\Phi(-0.544) \approx 0.2946, \quad \Phi(-0.744) \approx 0.2296$$
$$\mathrm{Bl}_A = 0.030769(0.2946) - 0.035(0.2296) = 0.00102$$
$$\mathrm{CplA}_0 = 0.975 \times 0.5 \times 0.00102 = 0.00050$$

**Caplet B ($[1.0, 1.5]$):** $F_2 = 4.19\% > K$, $v = 0.20$

$$\ln(F/K) = \ln(1.1967) = 0.1795$$
$$d_1 = \frac{0.1795 + 0.02}{0.20} = 1.00, \quad d_2 = 0.80$$
$$\Phi(1.00) = 0.8413, \quad \Phi(0.80) = 0.7881$$
$$\mathrm{Bl}_B = 0.041885(0.8413) - 0.035(0.7881) = 0.00766$$
$$\mathrm{CplB}_0 = 0.955 \times 0.5 \times 0.00766 = 0.00366$$

**Cap price:**

$$\boxed{\mathrm{Cap}_0 = 0.00050 + 0.00366 = 0.00416}$$

### Example 4: Swaption Pricing

**Payer swaption** expiring at $T_1 = 0.5$ into swap $[0.5, 1.5]$

**Annuity:**
$$C_{1,3}(0) = 0.5 \times P(0, 1.0) + 0.5 \times P(0, 1.5) = 0.5(0.975 + 0.955) = 0.965$$

**Swap rate:**
$$S = \frac{P(0, 0.5) - P(0, 1.5)}{C_{1,3}(0)} = \frac{0.990 - 0.955}{0.965} = \frac{0.035}{0.965} = 0.03627$$

**ATM swaption** ($K = S$), $v = 0.20$:

$$d_1 = 0.10, \quad d_2 = -0.10$$
$$\mathrm{Bl} = S \times 0.0796 = 0.03627 \times 0.0796 = 0.002887$$

**Swaption price:**
$$\boxed{\mathrm{Swpt}_0 = C_{1,3}(0) \times \mathrm{Bl} = 0.965 \times 0.002887 = 0.00279}$$

### Example 5: Par Swap Identity Verification

Using Example 4 data:

**Par swap rate:** $K_{\text{par}} = S = 0.03627$

**Verify PV equality:**

$$K_{\text{par}} \times C_{1,3}(0) = 0.03627 \times 0.965 = 0.0350$$

$$P(0, 0.5) - P(0, 1.5) = 0.990 - 0.955 = 0.0350 \checkmark$$

**Units:** $K$ is 1/year, $C$ is years, so $KC$ is dimensionless PV.

### Example 6: Coterminal Swaptions (Bermudan Setup)

Consider swaptions ending at $T_3 = 1.5$:

| Swaption | Expiry | Underlying Swap | Annuity |
|----------|--------|-----------------|---------|
| 6M×1Y | 0.5 | [0.5, 1.5] | $C_{1,3} = 0.965$ |
| 1Y×6M | 1.0 | [1.0, 1.5] | $C_{2,3} = 0.5 \times 0.955 = 0.4775$ |

**Swap rates:**

$$S_{1,3}(0) = \frac{0.990 - 0.955}{0.965} = 3.627\%$$

$$S_{2,3}(0) = \frac{0.975 - 0.955}{0.4775} = \frac{0.020}{0.4775} = 4.188\%$$

**Bermudan interpretation:** A Bermudan with exercise at 0.5 and 1.0 compares:
- At $T_1 = 0.5$: exercise value is swaption on [0.5, 1.5]
- At $T_2 = 1.0$: exercise value is swaption on [1.0, 1.5]

Calibrating to both coterminal swaptions ensures correct boundary conditions.

### Example 7: Calibration by Bisection

**Goal:** Find $\sigma$ so ATM caplet price = 0.001194 (from Example 2)

**ATM formula:**
$$\mathrm{Cpl}_0(\sigma) = P(0, T_2) \cdot \delta \cdot F \cdot [\Phi(d_1) - \Phi(d_2)]$$

where $d_1 = \frac{1}{2}\sigma\sqrt{T}$, $d_2 = -\frac{1}{2}\sigma\sqrt{T}$.

**Bracket:** $\sigma_L = 0.01$ gives price $\approx 0.00004$; $\sigma_H = 1.00$ gives price $\approx 0.00411$

**Iteration 1:** $\sigma_M = 0.505$, $v = 0.357$, price $\approx 0.00214$ (too high) → $\sigma_H = 0.505$

**Iteration 2:** $\sigma_M = 0.2575$, $v = 0.182$, price $\approx 0.00108$ (too low) → $\sigma_L = 0.2575$

**Converged:** $\sigma \in [0.2575, 0.505]$, true value $\approx 0.283$

### Example 8: Correlation Effect on Basket Variance

**Setup:** Two forwards with $\sigma_1 = 1\%$, $\sigma_2 = 1.5\%$, weights $w_1 = 0.6$, $w_2 = 0.4$

**Variance of weighted sum:**
$$\text{Var}(Y) = w_1^2\sigma_1^2 + w_2^2\sigma_2^2 + 2w_1w_2\rho\sigma_1\sigma_2$$

**Compute fixed terms:**
- $w_1^2\sigma_1^2 = 0.36 \times 0.0001 = 0.000036$
- $w_2^2\sigma_2^2 = 0.16 \times 0.000225 = 0.000036$
- $2w_1w_2\sigma_1\sigma_2 = 2 \times 0.24 \times 0.00015 = 0.000072$

**If $\rho = 0$:** Var = 0.000072, Std = 0.849%

**If $\rho = 0.8$:** Var = 0.000072 + 0.8(0.000072) = 0.0001296, Std = 1.139%

**Takeaway:** Higher correlation increases basket volatility → higher swaption prices.

### Example 9: Zero Volatility Limit

**Caplet B (OTM):** $F = 4.19\%$, $K = 3.5\%$, $\sigma = 0$

$$\mathrm{Cpl}_0(\sigma=0) = P(0, 1.5) \cdot \delta \cdot \max(F - K, 0) = 0.955 \times 0.5 \times 0.00689 = 0.00329$$

Compare to $\sigma > 0$: price was 0.00366. Option value exceeds intrinsic by time value.

**ATM swaption:** $S = K$, intrinsic = 0

$$\mathrm{Swpt}_0(\sigma=0) = 0$$

With $\sigma > 0$: price was 0.00279. The entire value is time value.

### Example 10: Drift Coupling Computation

**Setup:** Simulate $F_0$ under terminal measure $\mathbb{Q}^3$ with:
- $F_0(0) = 2.02\%$, $F_1(0) = 3.08\%$, $F_2(0) = 4.19\%$
- $\sigma_0 = \sigma_1 = \sigma_2 = 20\%$
- $\rho_{01} = \rho_{02} = \rho_{12} = 0.9$
- $\delta = 0.5$

**Drift of $F_0$ at $t = 0$:**

$$\mu_0^{(3)} = -\sigma_0 F_0 \left[\frac{\delta_1 F_1 \sigma_1 \rho_{01}}{1+\delta_1 F_1} + \frac{\delta_2 F_2 \sigma_2 \rho_{02}}{1+\delta_2 F_2}\right]$$

**Compute each term:**

Term 1: $\frac{0.5 \times 0.0308 \times 0.20 \times 0.9}{1 + 0.5 \times 0.0308} = \frac{0.00277}{1.0154} = 0.00273$

Term 2: $\frac{0.5 \times 0.0419 \times 0.20 \times 0.9}{1 + 0.5 \times 0.0419} = \frac{0.00377}{1.0209} = 0.00369$

**Total drift coefficient:**
$$\mu_0^{(3)} = -0.20 \times 0.0202 \times (0.00273 + 0.00369) = -0.0040 \times 0.00642 = -0.0000257$$

This is a small negative drift (about -2.6 bp per year), showing how terminal measure simulation pulls early forwards downward.

### Example 11: CMS Convexity Adjustment (Replication Intuition)

A CMS rate is a swap rate paid at a single date, not as a stream. The convexity adjustment arises because:

$$\mathbb{E}^{T_{\text{pay}}}[S_{\alpha,\beta}(T_\alpha)] \neq S_{\alpha,\beta}(0)$$

**Approximate adjustment (from Andersen-Piterbarg):**

For a swap rate with volatility $\sigma_S$ and duration $D$:

$$\text{Convexity adjustment} \approx \frac{\sigma_S^2 T D S}{1 + DS}$$

**Example:** $S = 3.5\%$, $\sigma_S = 15\%$, $T = 1$, $D = 4$ years

$$\text{Adj} \approx \frac{(0.15)^2 \times 1 \times 4 \times 0.035}{1 + 4 \times 0.035} = \frac{0.0225 \times 0.14}{1.14} = \frac{0.00315}{1.14} \approx 0.28\%$$

So the CMS rate expectation is about $3.5\% + 0.28\% = 3.78\%$—higher than the forward swap rate.

### Example 12: Swaption Cube Point via Monte Carlo

**Goal:** Price 1Y×2Y payer swaption at strike 4% using LMM simulation

**Setup:**
- Tenor grid: 0, 1, 2, 3 years ($\delta = 1$)
- Initial forwards: $F_0 = 3\%$, $F_1 = 3.5\%$, $F_2 = 4\%$
- Vols: 20% each
- Correlation: 0.85 between adjacent, 0.70 otherwise

**Step 1:** Simulate forwards to $T_1 = 1$ under terminal measure $\mathbb{Q}^3$

**Step 2:** At $T_1$, compute:
- Bond prices: $P(1, T_i) = \prod_{j}(1 + \delta F_j(1))^{-1}$
- Swap rate: $S_{1,3}(1) = \frac{P(1,1) - P(1,3)}{P(1,2) + P(1,3)}$
- Annuity: $C_{1,3}(1) = P(1,2) + P(1,3)$

**Step 3:** Payoff = $\max(S_{1,3}(1) - 0.04, 0) \times C_{1,3}(1)$

**Step 4:** Discount: $\text{PV} = P(0, 3) \times \mathbb{E}^3[\text{Payoff}]$

(With 10,000 paths and proper antithetic variance reduction, this converges to analytical approximation within ~1% typically.)

### Example 13: Rebonato Swaption Volatility Approximation

**Goal:** Compute the implied swaption volatility for a 6M×1Y swaption using Rebonato's approximation.

**Setup:** Using data from Example 1 and 4:
- Swap over $[T_1, T_3] = [0.5, 1.5]$ with two periods
- Forwards: $F_1(0) = 3.08\%$, $F_2(0) = 4.19\%$
- Volatilities: $\sigma_1 = \sigma_2 = 20\%$ (constant)
- Correlation: $\rho_{1,2} = 0.85$
- Accruals: $\delta_1 = \delta_2 = 0.5$
- Discount factors: $P(0, 1.0) = 0.975$, $P(0, 1.5) = 0.955$
- Swap rate: $S_{1,3}(0) = 3.627\%$ (from Example 4)
- Expiry: $T_1 = 0.5$

**Step 1: Compute annuity and weights**

$$C_{1,3}(0) = 0.5 \times 0.975 + 0.5 \times 0.955 = 0.965$$

$$w_1 = \frac{\delta_1 P(0, T_1)}{C_{1,3}(0)} = \frac{0.5 \times 0.975}{0.965} = 0.505$$

$$w_2 = \frac{\delta_2 P(0, T_2)}{C_{1,3}(0)} = \frac{0.5 \times 0.955}{0.965} = 0.495$$

Check: $w_1 + w_2 = 1.0$ ✓

**Step 2: Compute the double sum**

With constant volatilities, $\int_0^{T_1} \sigma_i \sigma_j dt = \sigma_i \sigma_j T_1 = 0.20 \times 0.20 \times 0.5 = 0.02$ for all pairs.

The variance formula becomes:
$$\left(v_{1,3}^{LMM}\right)^2 = \frac{T_1}{S_{1,3}(0)^2} \sum_{i,j=1}^{2} w_i w_j F_i(0) F_j(0) \rho_{i,j} \sigma_i \sigma_j$$

Expand the sum:
- $(i=1, j=1)$: $w_1^2 F_1^2 \rho_{11} \sigma_1^2 = (0.505)^2 (0.0308)^2 (1) (0.04) = 9.67 \times 10^{-6}$
- $(i=1, j=2)$: $w_1 w_2 F_1 F_2 \rho_{12} \sigma_1 \sigma_2 = (0.505)(0.495)(0.0308)(0.0419)(0.85)(0.04) = 1.10 \times 10^{-5}$
- $(i=2, j=1)$: Same as $(1,2)$ = $1.10 \times 10^{-5}$
- $(i=2, j=2)$: $w_2^2 F_2^2 \rho_{22} \sigma_2^2 = (0.495)^2 (0.0419)^2 (1) (0.04) = 1.72 \times 10^{-5}$

Sum = $9.67 \times 10^{-6} + 2 \times 1.10 \times 10^{-5} + 1.72 \times 10^{-5} = 4.89 \times 10^{-5}$

**Step 3: Compute swaption variance and volatility**

$$\left(v_{1,3}^{LMM}\right)^2 = \frac{0.5}{(0.03627)^2} \times 4.89 \times 10^{-5} = \frac{0.5 \times 4.89 \times 10^{-5}}{1.32 \times 10^{-3}} = 0.0186$$

$$v_{1,3}^{LMM} = \sqrt{0.0186} = 0.136$$

This is the total volatility. The annualized implied volatility is:

$$\sigma_{swpt}^{impl} = \frac{v}{\sqrt{T}} = \frac{0.136}{\sqrt{0.5}} = 0.193 = \boxed{19.3\%}$$

**Sanity check:** The swaption implied vol (19.3%) is close to but slightly below the forward vols (20%). The reduction reflects diversification: the swap rate is a weighted average of two forwards, and imperfect correlation ($\rho = 0.85 < 1$) reduces variance.

---

## A4.9 Summary

### Key Takeaways

1. **Market models** target market-quoted rates to align with Black quoting conventions
2. **LMM** models forward rates; each $F_i$ is lognormal under $\mathbb{Q}^{i+1}$, making caplets exactly Black
3. **Swap Market Model** models swap rates; $S_{\alpha,\beta}$ is lognormal under $\mathbb{Q}^{\alpha,\beta}$, making swaptions exactly Black
4. **Coupled drift** is the price of convenience—under a common measure, drifts depend on all forward levels (negative under terminal measure, positive under spot measure)
5. **Rebonato's approximation** links LMM forward parameters to swaption volatilities via a weighted covariance formula
6. **Calibration** balances caplet fit (volatility levels) and swaption fit (correlation structure)
7. **Regularization** is essential to avoid ringing and ensure hedge stability
8. **Coterminal swaptions** are critical for Bermudan pricing
9. **Two-curve framework** (distinct discount and index curves) is now standard for non-collateralized trades
10. **SOFR adaptation** requires reinterpreting the timing but preserves the framework

### Cheat Sheet

| Object | Definition |
|--------|------------|
| $F_i(t)$ | $\frac{1}{\delta_i}(P(t,T_i)/P(t,T_{i+1}) - 1)$ |
| $C_{\alpha,\beta}(t)$ | $\sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$ |
| $S_{\alpha,\beta}(t)$ | $(P(t,T_\alpha) - P(t,T_\beta))/C_{\alpha,\beta}(t)$ |
| $w_i(t)$ | $\delta_i P(t, T_i) / C_{\alpha,\beta}(t)$ (swap rate weights) |
| LMM dynamics | $dF_i = \sigma_i F_i dZ^{i+1}$ under $\mathbb{Q}^{i+1}$ |
| Terminal drift | $\mu_k^{(n)} = -\sigma_k F_k \sum_{j>k}\frac{\delta_j F_j \sigma_j \rho_{kj}}{1+\delta_j F_j}$ |
| Spot drift | $\mu_k^{(d)} = +\sigma_k F_k \sum_{j \le k}\frac{\delta_j F_j \sigma_j \rho_{kj}}{1+\delta_j F_j}$ |
| Rebonato approx | $(v_{\alpha,\beta}^{LMM})^2 \approx \sum_{i,j} \frac{w_i w_j F_i F_j \rho_{ij}}{S^2} \int_0^T \sigma_i \sigma_j dt$ |

---

## A4.10 Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $F_i(t)$? | $\frac{1}{\delta_i}(P(t,T_i)/P(t,T_{i+1}) - 1)$ |
| 2 | Under which measure is $F_i$ driftless in LMM? | $\mathbb{Q}^{i+1}$ (the $T_{i+1}$-forward measure) |
| 3 | What is the LMM diffusion for $F_i$ under its forward measure? | $dF_i = \sigma_i F_i dZ^{i+1}$ |
| 4 | What is "coupled drift"? | Under other measures, drift of $F_k$ depends on other forwards |
| 5 | Define swap annuity $C_{\alpha,\beta}(t)$ | $\sum_{k=\alpha+1}^{\beta} \delta_k P(t, T_k)$ |
| 6 | Define swap rate $S_{\alpha,\beta}(t)$ | $(P(t,T_\alpha) - P(t,T_\beta))/C_{\alpha,\beta}(t)$ |
| 7 | What numeraire gives the swap measure? | The swap annuity $C_{\alpha,\beta}(t)$ |
| 8 | What is payer swaption payoff at expiry? | $(S_{\alpha,\beta}(T_\alpha) - K)^+ \cdot C_{\alpha,\beta}(T_\alpha)$ |
| 9 | What is Black "$\mathrm{Bl}$"? | $\omega(F\Phi(\omega d_1) - K\Phi(\omega d_2))$ |
| 10 | In swap market model, what is lognormal? | The swap rate under the swap measure |
| 11 | Are forwards lognormal under risk-neutral measure? | No; they have state-dependent coupled drifts |
| 12 | What does "Black compatibility" mean? | Model produces Black prices under appropriate measure |
| 13 | Why add regularization in calibration? | To prevent ringing and improve stability |
| 14 | What's the warning about swaption matrices? | Stale entries cause calibration instability |
| 15 | Give an exponential correlation form | $\rho_{ij} = \exp(-\beta|T_i - T_j|)$ |
| 16 | What pins LMM volatility levels? | Caps/caplets |
| 17 | What pins LMM correlation? | Swaptions |
| 18 | What are coterminal swaptions? | Swaptions with same end date, different starts |
| 19 | Why are coterminals important? | Critical for Bermudan pricing |
| 20 | What is terminal measure $\mathbb{Q}^n$? | Measure with $P(t, T_n)$ as numeraire |
| 21 | Sign of drift under terminal measure? | Negative for earlier forwards |
| 22 | Sign of drift under spot measure? | Positive |
| 23 | What is the HJM drift restriction? | Volatility determines drift via no-arbitrage |
| 24 | How does LMM relate to HJM? | Discrete-tenor specialization |
| 25 | How does SOFR differ from LIBOR for LMM? | Backward-looking; known at period end |
| 26 | What is Rebonato's swaption volatility approximation? | $v^2 \approx \sum_{ij} w_i w_j F_i F_j \rho_{ij} \sigma_i \sigma_j T / S^2$ |
| 27 | What are swap rate weights $w_i$? | $w_i = \delta_i P(0, T_i) / C_{\alpha,\beta}(0)$; they sum to 1 |
| 28 | What is terminal correlation? | Correlation of forwards at option expiry, distinct from instantaneous |
| 29 | How does eigenvalue zeroing work? | Zero smallest eigenvalues, reconstruct matrix; ensures positive semi-definite |
| 30 | What is the two-curve framework? | Separate discount and index curves with cross-correlations |

---

## A4.11 Problem Set

**Problems 1-8 include solution sketches.**

1. **Given** $P(0,0) = 1$, $P(0, 0.5) = 0.99$, $\delta_0 = 0.5$, compute $F_0(0)$.

   **Sketch:** $F_0 = \frac{1}{0.5}(1/0.99 - 1) = 2 \times 0.0101 = 2.02\%$

2. **Compute** the annuity and par swap rate for a 1-year swap with semiannual payments, given $P(0, 0.5) = 0.99$, $P(0, 1.0) = 0.975$.

   **Sketch:** $A = 0.5(0.99 + 0.975) = 0.9825$. $S = (1 - 0.975)/0.9825 = 2.54\%$

3. **For an ATM caplet** with $F = K = 3\%$, $P(0, T) = 0.97$, $\delta = 0.5$, $T = 1$, $\sigma = 20\%$, compute $d_1$, $d_2$, and the caplet price.

   **Sketch:** $v = 0.20$, $d_1 = 0.10$, $d_2 = -0.10$. Price $= 0.97 \times 0.5 \times 0.03 \times 0.0796 = 0.00116$

4. **Explain** why caplets are "Black-compatible" in LMM.

   **Sketch:** Under $\mathbb{Q}^{i+1}$, $F_i$ is driftless lognormal, so caplet expectation is exactly Black.

5. **Explain** why swaptions are "Black-compatible" in the swap market model.

   **Sketch:** Under swap measure, $S_{\alpha,\beta}$ is lognormal; payoff is annuity times $(S-K)^+$, giving Black formula.

6. **What is "coupled drift"** in LMM and why does it arise?

   **Sketch:** Under non-canonical measures, drift of one forward depends on others via measure change (Girsanov). It arises from the product structure of bond ratios.

7. **Describe** the role of coterminal swaptions in Bermudan pricing.

   **Sketch:** Bermudan exercise decisions compare continuation vs immediate exercise. Immediate exercise values are coterminal swaption payoffs, so calibrating to these ensures correct boundaries.

8. **What is regularization** in LMM calibration and why is it needed?

   **Sketch:** Regularization adds penalty terms (smoothness, Tikhonov) to the objective to prevent ringing—oscillating calibrated volatilities from overfitting to noisy quotes.

9. Show that $C_{\alpha,\beta}(t)$ has units of years (PVBP interpretation).

10. Why is reduced-rank correlation computationally advantageous?

11. Describe one challenge in joint caps+swaptions calibration.

12. How can stale swaption matrix entries distort calibration?

13. If you increase forward-forward correlation, what happens to swaption prices?

14. Derive the drift of $F_{n-2}$ under the terminal measure $\mathbb{Q}^n$.

15. Explain conceptually why CMS rates have positive convexity adjustments.

16. Why might lognormal models be problematic when rates are negative?

---

## Source Map

### (A) Verified Facts — Source-Backed

| Fact | Source |
|------|--------|
| Forward rate definition from bond prices | Andersen & Piterbarg Vol 1; Brigo & Mercurio |
| Black function $\mathrm{Bl}$ definition | Andersen & Piterbarg; Brigo & Mercurio |
| LMM dynamics under forward measure | Andersen & Piterbarg Vol 2 Ch 14; Brigo & Mercurio Ch 6 |
| Coupled drift formulas (terminal/spot) | Andersen & Piterbarg Vol 2; Brigo & Mercurio Prop 6.3.1 (formulas 6.19, 6.20) |
| Drift sign explanation (Girsanov direction) | Brigo & Mercurio Ch 6 |
| Terminal and spot measure definitions | Andersen & Piterbarg Vol 2 |
| Swap annuity as numeraire | Andersen & Piterbarg; Brigo & Mercurio |
| Rebonato's swaption volatility approximation | Brigo & Mercurio Prop 6.15.1 (formula 6.67) |
| Terminal correlation formulas | Brigo & Mercurio formulas 6.70, 6.71 |
| Eigenvalue zeroing for rank reduction | Brigo & Mercurio Section 6.9 |
| Rebonato angles parameterization | Brigo & Mercurio Section 6.9 |
| Coterminal swaption calibration for Bermudans | Andersen & Piterbarg Vol 2 |
| Calibration as optimization with regularization | Andersen & Piterbarg Vol 2 |
| Ringing warnings | Andersen & Piterbarg Vol 2 |
| Tikhonov regularization | Andersen & Piterbarg Vol 2 |
| Correlation parameterizations | Andersen & Piterbarg Vol 2; Brigo & Mercurio |
| CMS convexity adjustment via replication | Andersen & Piterbarg Vol 2 |
| Predictor-corrector schemes | Andersen & Piterbarg Vol 2 |
| Two-curve framework (HJM/LMM extensions) | Andersen & Piterbarg Vol 2 Section 15.5 |
| LFM-LSM incompatibility | Brigo & Mercurio Section 6.10 |

### (B) Reasoned Inference — Derived from (A)

- Connection between LMM and HJM: derived from comparing discrete vs continuous tenor structures
- "Black compatibility" as design constraint: inferred from alignment between model measures and market conventions
- Swap rate as weighted average of forwards: derived from bond ratio decomposition
- SOFR adaptation approach: inferred from rate definition equivalence

### (C) Flagged Uncertainties

- **Exact numerical optimizer choice:** Sources discuss problem structure but don't mandate LM vs SQP vs gradient methods. I'm not sure which specific algorithm your desk prefers.
- **Full risk-neutral drift expression:** Depends on exact bank-account construction ($B(t)$ vs $B_d(t)$) and $\beta(t)$ definition not fully reproduced here.
- **Multi-curve depth:** How deep to integrate OIS discounting vs projection curve separation is desk-convention dependent.
- **SOFR timing adjustments:** Specific treatment of lookback periods and payment delays varies by contract; generic framework described but exact details not pinned.
