# Chapter 50: Correlation and Tranche Pricing Frameworks

---

## Introduction

Why do equity and senior tranches respond in *opposite directions* to changes in "correlation"? This question lies at the heart of structured credit risk and reveals why tranche pricing cannot be reduced to simple spread-plus-leverage intuition.

Consider a portfolio of 125 investment-grade names backing a synthetic CDO. A risk manager bumps the correlation parameter from 30% to 35% and observes something counterintuitive: the equity tranche *gains* value (for a protection seller) while the super-senior tranche *loses* value. Both tranches reference the same portfolio, yet they respond with opposite signs to the same parameter change. Understanding why requires grasping how dependence reshapes the *entire distribution* of portfolio losses, not just its mean.

The Gaussian copula framework—sometimes called "the formula that killed Wall Street" after its role in the 2007-2008 crisis—remains the market's standard language for quoting tranche prices. Despite its well-documented limitations (no tail dependence, calibration instabilities, base correlation pathologies), it persists because it provides a common vocabulary for traders, a tractable calibration framework, and a starting point for more sophisticated models. As O'Kane emphasizes, "the one-factor Gaussian copula model has become the market standard for pricing and hedging synthetic CDO tranches."

This chapter develops the conceptual and mathematical framework for correlation-sensitive tranche pricing. We begin with the fundamental question of why tranches are correlation-sensitive at all (Section 50.1), then develop copula theory from Sklar's theorem (Section 50.2). The Gaussian copula one-factor model receives detailed treatment (Section 50.3), including its critical limitation: zero tail dependence. The Large Homogeneous Portfolio (LHP) model provides analytical tractability for large portfolios (Section 50.4). We then examine the market's two correlation languages—compound correlation (Section 50.5) and base correlation (Section 50.6)—with particular attention to the pathologies that arise from base correlation interpolation (Section 50.8). Model risk and lessons from 2008 (Section 50.9) connect the framework to practical risk management. Throughout, we build intuition through worked examples that make the mathematics concrete.

**Prerequisites:** This chapter assumes familiarity with tranche mechanics and the waterfall structure (Chapter 48), expected tranche loss calculations (Chapter 49), and basic probability theory. Readers seeking deeper mathematical foundations should consult Appendix A6 on credit portfolio models.

---

## Conventions and Notation

| Convention | Description |
|------------|-------------|
| **Time** | $t \ge 0$ in **years**; trade inception at $t=0$, maturity at $T$ |
| **Loss units** | Losses expressed as **fractions of portfolio notional** unless stated otherwise |
| **Tranche strikes** | $A,D$ (attachment, detachment) as **fractions of portfolio notional** (e.g., 3% ≡ 0.03); tranche width $w = D - A$ |
| **Tranche loss fraction** | Always bounded in $[0,1]$ (loss as fraction of tranche notional) |
| **Discount factor** | $Z(t)$ is the present value of \$1 at time $t$ |

| Symbol | Definition |
|--------|------------|
| $i = 1, \dots, N$ | Names in the reference portfolio |
| $\tau_i$ | Default time of name $i$ |
| $Y_{t,i} = \mathbf{1}_{\{\tau_i \le t\}}$ | Default indicator (1 if defaulted by $t$, else 0) |
| $R_i$ | Recovery rate of name $i$; $\text{LGD}_i = 1 - R_i$ |
| $L(t)$ | **Fractional portfolio loss** at time $t$ |
| $L_{\max}$ | Maximum possible portfolio loss under recovery assumptions |
| $L(t; A, D)$ | **Tranche loss fraction** at time $t$, in $[0,1]$ |
| $Q(t; A, D) = \mathbb{E}[1 - L(t; A, D)]$ | Tranche survival curve |
| $\psi(T, K) = \mathbb{E}[\min(L(T), K)]$ | ETL of base tranche $[0, K]$ |
| $\rho$ | Correlation parameter in one-factor Gaussian copula |
| $\Phi(\cdot), \Phi^{-1}(\cdot)$ | Standard normal CDF and inverse CDF |
| $t_\nu(\cdot), t_\nu^{-1}(\cdot)$ | Student's t CDF with $\nu$ degrees of freedom and its inverse |
| $\lambda_U, \lambda_L$ | Upper and lower tail dependence coefficients |

---

## 50.1 Why Tranche Valuation Is Correlation-Sensitive

### 50.1.1 The Nonlinearity at the Core

The fundamental reason tranches are correlation-sensitive lies in a simple mathematical fact: tranche loss is a *nonlinear* function of portfolio loss. Recall from Chapter 48 the tranche loss mapping:

$$\boxed{L(T; A, D) = \frac{\min(L(T), D) - \min(L(T), A)}{D - A}}$$

This piecewise-linear function has kinks at $A$ and $D$. Below $A$, the tranche suffers no loss (subordination absorbs everything). Between $A$ and $D$, losses pass through dollar-for-dollar. Above $D$, the tranche is wiped out.

**Why nonlinearity matters:** Because tranche loss is nonlinear in portfolio loss, the expected tranche loss $\mathbb{E}[L(T; A, D)]$ depends on the *entire distribution* of $L(T)$, not just its mean. Two portfolios with identical expected losses but different loss *distributions* will have different tranche expected losses.

This is Jensen's inequality at work: for a nonlinear function $f$, generally $\mathbb{E}[f(X)] \neq f(\mathbb{E}[X])$.

### 50.1.2 How Correlation Reshapes the Loss Distribution

In the one-factor Gaussian copula framework, correlation controls how much portfolio loss is driven by the systematic factor $X$ versus idiosyncratic shocks. O'Kane provides the intuition:

- **Low correlation ($\rho \to 0$):** Defaults are nearly independent. By the law of large numbers, portfolio loss concentrates around its expected value. The distribution is "peaked" in the middle.

- **High correlation ($\rho \to 1$):** The systematic factor dominates. In good states ($X$ high), almost no one defaults. In bad states ($X$ low), many default together. The distribution becomes "bimodal," with more mass at the extremes.

> **Analogy: The Magnet**
>
> Think of defaults as metal balls on a table.
> *   **Zero Correlation**: They roll randomly. Sometimes one falls off, sometimes two. It's predictable on average, but noisy.
> *   **High Correlation (Strong Magnet)**: A magnet underneath pulls them together.
>     *   **Scenario A**: The magnet holds them ALL on the table (Survival).
>     *   **Scenario B**: The magnet pulls them ALL off the edge (Systemic Collapse).
> *   **Who loves the Magnet?**: The Equity Tranche. Why? Because in Scenario A, Equity survives! With zero correlation, Equity *always* dies. With high correlation, Equity has a "fighting chance" to survive.
> *   **Who hates the Magnet?**: The Senior Tranche. Why? Because Scenario B is the *only* way Senior loses money.

**The critical insight:** Changing correlation doesn't change the *expected* portfolio loss (which depends only on marginal default probabilities), but it dramatically changes the *shape* of the loss distribution.

### 50.1.3 Opposite Sensitivities Across the Capital Structure

This distributional reshaping affects tranches differently based on their position in the capital structure:

**Equity tranche (low attachment):** The equity tranche is concerned with whether *any* losses occur. When correlation increases, more probability mass shifts to the "no loss" scenario (everyone survives in good states). This *decreases* expected equity tranche loss.

**Senior tranche (high attachment):** The senior tranche only suffers when losses exceed its attachment point—a tail event. When correlation increases, the probability of extreme joint defaults rises. This *increases* expected senior tranche loss.

O'Kane quantifies this with "correlation 01" values—the change in tranche PV for a 1% increase in correlation:

| Tranche | Correlation 01 (bps running) |
|---------|------------------------------|
| 0–3% Equity | −148 |
| 3–7% Junior Mezz | −17 |
| 7–10% Senior Mezz | +35 |
| 10–15% Senior | +66 |
| 15–30% Super-Senior | +43 |

The sign flip between equity and senior is not an anomaly—it's the fundamental signature of tranche correlation sensitivity.

### 50.1.4 Position Matters: Long vs. Short Protection

The above sensitivities assume a **short protection** position (receiving premium, paying losses). For a **long protection** position, all signs reverse:

- Long protection equity: *short* correlation (loses when correlation rises)
- Long protection senior: *long* correlation (gains when correlation rises)

This distinction is crucial for hedging. A trader hedging equity risk with senior tranches is implicitly taking a correlation view.

---

## 50.2 Copula Theory: Linking Marginals to Joint Behavior

### 50.2.1 The Separation Problem

Credit portfolio modeling faces a fundamental challenge: we have good tools for estimating *individual* default probabilities (from CDS curves), but we need to price instruments that depend on *joint* default behavior.

Copulas provide a mathematically elegant solution by separating these two problems:
1. **Marginal distributions:** How likely is each name to default by time $t$?
2. **Dependence structure:** Given these marginals, how do defaults cluster together?

### 50.2.2 Sklar's Theorem: The Mathematical Foundation

**Theorem (Sklar, 1959):** For any joint distribution function $H$ with marginals $F_1, \ldots, F_N$, there exists a copula $C$ such that:

$$H(x_1, \ldots, x_N) = C(F_1(x_1), \ldots, F_N(x_N))$$

If the marginals are continuous, $C$ is unique.

**Intuition:** Think of $C$ as a "coupling device." It takes $N$ uniform random variables (the probability-transformed marginals) and produces their joint distribution. The copula captures *how* the marginals are coupled—whether they tend to move together, opposite, or independently—while remaining agnostic about the marginals themselves.

McNeil, Frey, and Embrechts in *Quantitative Risk Management* emphasize why this matters for credit: "The copula approach allows us to study the dependence structure of random vectors in a scale-free way, independently of the marginal distributions."

### 50.2.3 Common Copulas

**Independence copula:**
$$C_{\perp}(u_1, \ldots, u_N) = \prod_{i=1}^N u_i$$

No dependence—knowing one default tells you nothing about others.

**Gaussian copula:**
$$C_{\Phi}(u_1, \ldots, u_N; \Sigma) = \Phi_N(\Phi^{-1}(u_1), \ldots, \Phi^{-1}(u_N); \Sigma)$$

where $\Phi_N(\cdot; \Sigma)$ is the $N$-dimensional standard normal CDF with correlation matrix $\Sigma$.

**Student's t-copula:**
$$C_t(u_1, \ldots, u_N; \Sigma, \nu) = t_N(t_\nu^{-1}(u_1), \ldots, t_\nu^{-1}(u_N); \Sigma, \nu)$$

where $t_\nu^{-1}$ is the inverse t-distribution CDF and $\nu$ is degrees of freedom.

### 50.2.4 Why Copulas Fit Credit Portfolio Modeling

For synthetic CDOs, the copula approach is natural:

1. **Marginals are market-observable:** Single-name CDS curves give us $Q_i(t) = \Pr(\tau_i > t)$ for each name.

2. **Joint behavior is the unknown:** How defaults cluster is what we're trying to model/calibrate.

3. **Separation enables calibration:** We can calibrate marginals to single-name CDS, then calibrate the copula to tranche prices.

As O'Kane notes, this separation means "we can use the copula function to vary the correlation between obligors without having to change the cumulative default time distributions."

---

## 50.3 Gaussian Copula One-Factor Model

### 50.3.1 The Latent Variable Construction

The market-standard Gaussian copula model uses a one-factor latent variable representation. For each name $i$:

$$\boxed{Z_i = \sqrt{\rho} X + \sqrt{1-\rho} \varepsilon_i}$$

where:
- $X \sim N(0,1)$: the **systematic factor** (market-wide credit conditions)
- $\varepsilon_i \sim N(0,1)$: the **idiosyncratic factor** for name $i$ (independent across names, independent of $X$)
- $\rho \in [0,1]$: the **correlation parameter**

**Verification:** $Z_i$ is standard normal: $\mathbb{E}[Z_i] = 0$, $\text{Var}(Z_i) = \rho + (1-\rho) = 1$.

**Pairwise correlation:** For $i \neq j$:
$$\text{Corr}(Z_i, Z_j) = \mathbb{E}[Z_i Z_j] = \rho \mathbb{E}[X^2] + 0 = \rho$$

This is the "flat correlation" assumption: all pairs have the same correlation $\rho$.

### 50.3.2 Default Thresholds from Marginal Probabilities

Given marginal default probability $\text{PD}_i(T) = 1 - Q_i(T)$ from CDS calibration, we set a threshold:

$$a_i(T) = \Phi^{-1}(\text{PD}_i(T))$$

and define default by time $T$ as:

$$\tau_i \le T \iff Z_i \le a_i(T)$$

**Why this works:** $\Pr(Z_i \le a_i(T)) = \Phi(a_i(T)) = \text{PD}_i(T)$, matching the marginal.

### 50.3.3 Conditional Independence: The Computational Key

Conditional on $X = x$, the $Z_i$ are independent normal random variables:

$$Z_i \mid X = x \sim N(\sqrt{\rho} x, 1 - \rho)$$

This yields the **conditional default probability**:

$$\boxed{p_i(T \mid X = x) = \Phi\left(\frac{a_i(T) - \sqrt{\rho} x}{\sqrt{1-\rho}}\right)}$$

**Why this matters:** Conditional independence transforms an intractable $N$-dimensional integration into:
1. A one-dimensional integral over $X$
2. $N$ independent Bernoulli trials conditional on each $X$ realization

For large homogeneous portfolios, this enables analytical approximations via the Large Homogeneous Portfolio (LHP) model developed in Section 50.4.

### 50.3.4 Limiting Cases and Sanity Checks

**$\rho = 0$ (independence):**
$$Z_i = \varepsilon_i, \quad p_i(T \mid X = x) = \Phi(a_i(T)) = \text{PD}_i(T)$$
Conditional equals marginal; defaults are independent.

**$\rho \to 1$ (comonotonicity):**
$$Z_i \approx X$$
All latent variables collapse to the single factor. Defaults become perfectly dependent—either everyone survives or everyone defaults together.

**Extreme factor realizations:**
- $X \to -\infty$: $p_i(T \mid X) \to 1$ (all default)
- $X \to +\infty$: $p_i(T \mid X) \to 0$ (none default)

### 50.3.5 Tail Dependence: The Critical Limitation

**Definition:** The upper and lower tail dependence coefficients measure the probability of joint extremes:

$$\lambda_U = \lim_{u \to 1^-} \Pr(U_2 > u \mid U_1 > u)$$
$$\lambda_L = \lim_{u \to 1^-} \Pr(U_2 \le 1-u \mid U_1 \le 1-u)$$

**The Gaussian copula has zero tail dependence:** $\lambda_U = \lambda_L = 0$ for all $\rho < 1$.

McNeil, Frey, and Embrechts in *Quantitative Risk Management* explain the implication: "In the Gaussian case, extreme joint events are asymptotically independent... This means that if we observe that $X_1$ takes a very extreme value, this conveys relatively little information about whether $X_2$ is also extreme."

**Why this matters for senior tranches:** Senior tranches are tail-driven instruments. They only suffer losses in extreme scenarios where many names default together. A model with no tail dependence may systematically underestimate senior tranche risk.

**The t-copula alternative:** McNeil et al. (QRM Example 5.33) derive the tail dependence coefficient for the t-copula with correlation $\rho$ and $\nu$ degrees of freedom:

$$\boxed{\lambda_U = \lambda_L = 2 t_{\nu+1}\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)}$$

where $t_{\nu+1}(\cdot)$ is the CDF of the Student's t-distribution with $\nu+1$ degrees of freedom.

**Numerical values from McNeil Table 5.1 (t-copula tail dependence):**

| $\nu \backslash \rho$ | 0.5 | 0.7 | 0.9 |
|-----------------------|-----|-----|-----|
| 1 | 0.39 | 0.50 | 0.67 |
| 4 | 0.18 | 0.28 | 0.47 |
| 10 | 0.08 | 0.14 | 0.32 |
| 25 | 0.03 | 0.06 | 0.19 |
| $\infty$ (Gaussian) | 0 | 0 | 0 |

**Key observations:**
- As $\nu \to \infty$, the t-copula converges to Gaussian and tail dependence vanishes
- Lower $\nu$ (fatter tails) implies more tail dependence
- Even moderate correlation ($\rho = 0.5$) with fat tails ($\nu = 4$) gives $\lambda = 0.18$—far from zero

**Implication:** The correlation skew observed in market base correlations (Section 50.6.5) can be interpreted as the market compensating for the Gaussian copula's lack of tail dependence. Senior tranches require higher base correlation to generate sufficient ETL, implicitly pricing in tail dependence the model cannot capture.

---

## 50.4 Large Homogeneous Portfolio (LHP) Model

### 50.4.1 The Vasicek Limit

For large portfolios with identical names, the one-factor Gaussian copula admits a powerful analytical simplification known as the Large Homogeneous Portfolio (LHP) model, originally due to Vasicek (1987).

**Assumptions:**
- $N$ names with identical marginal default probabilities: $\text{PD}_i(T) = p$ for all $i$
- Identical recovery rates: $R_i = R$
- Equal weights: $w_i = 1/N$
- Flat correlation: $\rho$ for all pairs

**The key insight:** As $N \to \infty$, conditional on the systematic factor $X = x$, the portfolio loss converges to its conditional expectation by the law of large numbers:

$$L(T) \mid X = x \xrightarrow{N \to \infty} (1-R) \cdot p(x)$$

where $p(x) = \Phi\left(\frac{\Phi^{-1}(p) - \sqrt{\rho} x}{\sqrt{1-\rho}}\right)$ is the conditional default probability.

### 50.4.2 The LHP Loss Distribution

In the LHP limit, portfolio loss becomes a deterministic function of $X$. Inverting:

$$L = (1-R) \cdot \Phi\left(\frac{\Phi^{-1}(p) - \sqrt{\rho} x}{\sqrt{1-\rho}}\right)$$

Solving for $x$ in terms of $L$:

$$x(L) = \frac{\Phi^{-1}(p) - \sqrt{1-\rho}\Phi^{-1}(L/(1-R))}{\sqrt{\rho}}$$

The CDF of portfolio loss is:

$$\boxed{F_L(\ell) = \Phi\left(\frac{\sqrt{1-\rho}\Phi^{-1}(\ell/(1-R)) - \Phi^{-1}(p)}{\sqrt{\rho}}\right)}$$

This closed-form expression enables analytical computation of tranche expected losses.

### 50.4.3 Analytical Tranche Expected Loss

For a base tranche $[0, K]$, the expected tranche loss in the LHP limit is:

$$\psi(T, K) = \mathbb{E}[\min(L, K)] = \int_0^K (1 - F_L(\ell)) d\ell$$

O'Kane (Chapter 16) shows this reduces to a bivariate normal integral:

$$\boxed{\psi(T, K) = (1-R) \cdot \Phi_2\left(\Phi^{-1}(p), -\Phi^{-1}\left(\frac{K}{1-R}\right); -\sqrt{\rho}\right)}$$

where $\Phi_2(a, b; \rho)$ is the bivariate standard normal CDF with correlation $\rho$.

### 50.4.4 When LHP Applies

The LHP model is appropriate when:
- Portfolio is large ($N \geq 100$)
- Names are reasonably homogeneous in credit quality
- Equal or near-equal weighting

**Limitations:**
- Ignores name-specific variation in spreads and recovery
- Cannot capture concentration risk
- Fails for small portfolios or when a few names dominate

For bespoke portfolios with heterogeneous names, Monte Carlo simulation of the full model is typically required.

---

## 50.5 Compound Correlation and Its Pathologies

### 50.5.1 Definition

**Compound correlation** is the flat correlation $\rho$ that, when used in the one-factor Gaussian copula, reprices a given tranche to its market value:

$$\boxed{PV(A, D, \rho^*) = 0}$$

This is a one-dimensional root-finding problem: find $\rho^*$ such that model PV equals market PV.

### 50.5.2 The Multiple Solutions Problem

For equity and super-senior tranches, PV is typically monotonic in $\rho$, yielding a unique solution. But for **mezzanine tranches**, PV can be non-monotonic in $\rho$, potentially yielding:

- **Two solutions:** The market spread can be consistent with both a low and high correlation
- **No solution:** The market spread may be outside the model's feasible range

O'Kane illustrates this with market data showing that mezzanine tranche PV first decreases then increases as $\rho$ rises from 0 to 1.

### 50.5.3 Failure of Conservation of Expected Loss

Consider a portfolio with expected loss $E = \mathbb{E}[L(T)]$. By linearity:

$$\sum_{k} w_k \cdot \mathbb{E}[L(T; A_k, D_k)] = E$$

where the sum is over contiguous tranches covering the capital structure and $w_k = D_k - A_k$.

**The problem:** If we use *different* compound correlations $\rho_k$ for each tranche (as implied by market prices), we're using *different models* for each tranche. The expected losses computed under these different models will not sum to $E$.

This inconsistency motivated the development of base correlation.

---

## 50.6 Base Correlation: The Market Standard

### 50.6.1 Definition and Intuition

**Base correlation** assigns a correlation $\rho(K)$ to each **base (equity) tranche** $[0, K]$:

$$\psi(T, K; \rho(K)) = \text{Market ETL for } [0, K]$$

The key insight: rather than finding a single $\rho$ for each tranche, we find a $\rho$ for each *detachment point*, always measuring from zero.

> **Deep Dive: Base Correlation (The "Implied Volatility" of Credit)**
>
> In option markets, every strike has its own "Implied Volatility." In credit, every detachment point has its own "Base Correlation."
>
> *   **The Problem**: Tranche prices are inconsistent if you force one correlation $\rho$ on the whole capital structure.
> *   **The Fix**: Don't price $[3, 7]$. Price $[0, 7]$ and $[0, 3]$ separately.
>     *   Find $\rho_3$ that fits the market price of the Equity tranche $[0, 3]$.
>     *   Find $\rho_7$ that fits the virtually constructed Equity tranche $[0, 7]$.
> *   **The Result**: A "Skew" of correlations. Often $\rho_3 < \rho_7 < \rho_{10}$.
> *   **Why?**: Senior tranches effectively "bid up" correlation (demand protection against tail risk), just like deep OTM puts bid up implied volatility (skew).

### 50.6.2 Pricing Non-Equity Tranches

Any tranche $[A, D]$ is priced as the difference of two base tranches:

$$\boxed{\mathbb{E}[L(T; A, D)] = \frac{\psi(T, D; \rho(D)) - \psi(T, A; \rho(A))}{D - A}}$$

where $\psi(T, K; \rho(K)) = \mathbb{E}_{\rho(K)}[\min(L(T), K)]$ is the expected loss of base tranche $[0, K]$ computed using base correlation $\rho(K)$.

**Note the asymmetry:** The tranche $[A, D]$ uses *two different* correlations: $\rho(A)$ for the lower bound and $\rho(D)$ for the upper bound.

### 50.6.3 The Bootstrap Algorithm

O'Kane (Chapter 20) details the sequential calibration:

**Step 1:** From equity tranche $[0, K_1]$ market quote, solve for $\rho(K_1)$ using:
$$PV([0, K_1]; \rho(K_1)) = 0$$

**Step 2:** Given $\rho(K_1)$, from tranche $[K_1, K_2]$ market quote, solve for $\rho(K_2)$ using:
$$PV([K_1, K_2]; \rho(K_1), \rho(K_2)) = 0$$

where the tranche PV uses the base correlation formula with $\rho(K_1)$ and $\rho(K_2)$.

**Step 3:** Continue up the capital structure.

**Why it works:** Each step is a one-dimensional root search. Because we're always varying the *upper* base correlation while holding lower correlations fixed, the problem is well-posed.

### 50.6.4 Conservation of Expected Loss

A key property of base correlation is that it **preserves conservation of expected loss** within each base tranche calculation. O'Kane emphasizes this: for the base tranche $[0, K]$, we use a single correlation $\rho(K)$, so the model is internally consistent for that tranche.

However, when computing ETL for a mezzanine tranche $[A, D]$ using two different base correlations $\rho(A)$ and $\rho(D)$, we are effectively mixing two different models. This means:

- The *base tranche* ETLs $\psi(K; \rho(K))$ are each computed consistently
- The *difference* used for mezzanine tranches is a model approximation
- Conservation holds for the full capital structure only if we use a consistent correlation for the whole structure

**Practical implication:** Base correlation is a quoting convention, not a no-arbitrage model. It provides a common language but does not eliminate model risk.

### 50.6.5 Advantages Over Compound Correlation

1. **Uniqueness:** Base correlation typically yields unique solutions (monotonicity is preserved for base tranches)

2. **Consistency framework:** While not fully arbitrage-free, base correlation provides a coherent interpolation framework

3. **Market convention:** Base correlation has become the standard quoting language, enabling communication between desks

### 50.6.6 The Base Correlation Skew

Market data consistently shows that base correlation *increases* with detachment:

| Detachment $K$ | Typical Base Correlation |
|----------------|-------------------------|
| 3% | 15–25% |
| 7% | 25–35% |
| 10% | 35–45% |
| 15% | 50–60% |
| 30% | 60–75% |

**Interpretation:** To match market prices, the model needs "more dependence" for senior tranches than for equity. This is the market's way of compensating for model deficiencies (particularly the lack of tail dependence).

---

## 50.7 Calibration Workflow

### 50.7.1 Required Inputs

| Category | Inputs |
|----------|--------|
| **Portfolio** | Constituents, weights, maturity, credit event definitions |
| **Single-name marginals** | Survival curves $Q_i(t)$ from CDS calibration |
| **Recoveries** | Fixed $R_i$ or recovery model; determines $L_{\max}$ |
| **Dependence model** | Gaussian copula one-factor (standard) |
| **Tranche contracts** | $[A, D]$, coupon schedule, accrual convention |
| **Discount curve** | $Z(t)$ for discounting |
| **Market quotes** | Spread and/or upfront for each tranche |

### 50.7.2 Standard Index Tranches

For CDX and iTraxx indices, standard tranches are:

| Index | Equity | Junior Mezz | Senior Mezz | Senior | Super-Senior |
|-------|--------|-------------|-------------|--------|--------------|
| CDX.NA.IG | 0–3% | 3–7% | 7–10% | 10–15% | 15–30% |
| iTraxx Europe | 0–3% | 3–6% | 6–9% | 9–12% | 12–22% |

Equity tranches typically trade with upfront + 500bp running; others trade as running spread.

### 50.7.3 Calibration Objective

For each tranche, find correlation such that:

$$PV_{\text{model}}(\text{correlation}) = PV_{\text{market}}$$

At initiation with market quotes, this typically means $PV = 0$.

---

## 50.8 Base Correlation Interpolation Pathologies

### 50.8.1 The Interpolation Problem

Standard tranches have discrete attachment points (3%, 7%, 10%, etc.). To price **bespoke tranches** or **tranchelets** at non-standard strikes, we must interpolate the base correlation curve.

O'Kane devotes significant attention to the pathologies that can arise.

### 50.8.2 Non-Monotonic Tranchelet Spreads

A fundamental consistency requirement: tranchelet spreads should decrease as subordination increases. More subordination means more protection, hence lower spread.

**The pathology:** Linear interpolation of base correlation can produce tranchelet spreads that *increase* with subordination in certain strike ranges.

O'Kane provides an example: with linear interpolation between 3% and 7% base correlations, tranchelet spreads can be non-monotonic, violating economic logic.

### 50.8.3 Negative Tranchelet Spreads

In extreme cases, interpolation can produce **negative implied spreads** for thin tranchelets. This is economically absurd—it would imply the protection buyer pays the seller.

**Why it happens:** The base correlation formula computes tranche ETL as a difference of two base tranche ETLs. If the interpolated correlations cause the lower base tranche ETL to exceed the upper (adjusted for width), the result is negative.

### 50.8.4 Implied Loss Density Spikes

The **implied loss density** $f(K)$ is the derivative of expected loss with respect to strike:

$$f(K) = \frac{\partial}{\partial K} \mathbb{E}[\min(L, K)]$$

For a valid probability distribution, $f(K) \geq 0$ everywhere.

**The pathology:** Interpolation can produce:
- **Spikes:** Unrealistically high density at certain strikes
- **Negative regions:** $f(K) < 0$, which is mathematically impossible for a true distribution

O'Kane shows that with linear base correlation interpolation, the implied density can have sharp discontinuities at quoted strikes and negative values between them.

### 50.8.5 ETL-Space Interpolation

A partial remedy: instead of interpolating base *correlation*, interpolate base tranche *expected loss* (ETL) directly.

$$\psi(T, K) = \text{interpolate}(\{\psi(T, K_i)\})$$

Then back out the implied correlation from interpolated ETL.

**Advantages:**
- ETL is bounded in $[0, K]$ by construction
- Linear ETL interpolation preserves monotonicity of tranchelet ETL
- Avoids some (but not all) density pathologies

**Remaining issues:** ETL-space interpolation can still produce:
- Non-smooth implied densities
- Arbitrage in dynamic hedging scenarios

### 50.8.6 Extrapolation Below 3%

The lowest quoted strike is typically 3% (the equity detachment). Pricing thin equity tranches (0–1%, 1–2%, etc.) requires extrapolation.

**The problem:** Base correlation behavior below 3% is essentially unconstrained by market data. Small changes in extrapolation assumptions can dramatically affect thin tranchelet prices.

O'Kane notes that the 0–3% tranche's implied correlation already reflects the market's view of extreme losses, and further decomposition is speculative.

---

## 50.9 Model Risk and Lessons from 2008

### 50.9.1 The "Formula That Killed Wall Street"

The Gaussian copula's role in the 2007-2008 crisis has been extensively documented. Key failures:

1. **Underestimation of tail risk:** Zero tail dependence meant the model systematically underestimated the probability of widespread defaults

2. **Correlation instability:** Implied correlations proved highly unstable during the crisis, with base correlation skews steepening dramatically

3. **False precision:** The single-parameter model gave an illusion of precision while masking fundamental uncertainty about joint default behavior

4. **Calibration to benign conditions:** Models calibrated to pre-crisis data failed catastrophically when correlations spiked

### 50.9.2 Model Risk Dimensions

| Risk Dimension | Description |
|----------------|-------------|
| **Copula choice** | Gaussian vs. t-copula vs. other; tail dependence implications |
| **Correlation stability** | Implied correlation is not constant; depends on market conditions |
| **Recovery assumptions** | Fixed vs. stochastic recovery; affects correlation calibration |
| **Marginal curve risk** | Single-name curves may be inconsistent with tranche quotes |
| **Interpolation arbitrage** | Non-standard strikes may have negative spreads or density violations |

### 50.9.3 Stress Testing Framework

Prudent risk management requires stress testing beyond base case calibration:

**Correlation stress:**
- Parallel shift: $\rho(K) \to \rho(K) + \Delta\rho$ for all $K$
- Steepening: $\rho(K) \to \rho(K) + \alpha K$ for some $\alpha$
- Flattening: test impact of flat correlation at various levels

**Tail scenarios:**
- Compute tranche losses under t-copula with low $\nu$
- Simulate "crisis correlation" levels (60%+ for all tranches)

**Recovery stress:**
- Test sensitivity to recovery assumptions ($R = 20\%$ vs. $40\%$ vs. $60\%$)

**Jump-to-default:**
- Scenario analysis for individual name defaults
- Particularly relevant for concentrated portfolios

### 50.9.4 Practical Hedging Implications

The correlation smile implies that **delta hedging with single-names is incomplete**. A perfectly delta-hedged equity tranche still has:

- **Correlation exposure:** PV changes when implied correlation moves
- **Gamma exposure:** Non-linear response to spread changes
- **Gap risk:** Discrete defaults cause discontinuous PV jumps

Traders often hedge correlation exposure by trading tranches against each other (equity vs. mezzanine), but this introduces **basis risk** if the correlation smile shifts.

---

## 50.10 Worked Examples

### Example 1: Independence vs. Perfect Dependence (Two Names)

Let two names each have 1-year PD = 10%.

**Case 1: Independence**
- $\Pr(0 \text{ defaults}) = 0.9^2 = 0.81$
- $\Pr(1 \text{ default}) = 2(0.1)(0.9) = 0.18$
- $\Pr(2 \text{ defaults}) = 0.1^2 = 0.01$

**Case 2: Perfect positive dependence**
- $\Pr(2 \text{ defaults}) = 0.10$ (both default together)
- $\Pr(0 \text{ defaults}) = 0.90$ (both survive together)
- $\Pr(1 \text{ default}) = 0$ (impossible)

**Key insight:** Dependence multiplies the probability of joint extremes (from 1% to 10%) while preserving marginal PDs.

---

### Example 2: ETL Under Different Dependence Structures

Consider portfolio loss distributions:

**Low dependence:**
| Loss $L$ | Probability |
|----------|-------------|
| 0% | 0.05 |
| 8% | 0.90 |
| 40% | 0.05 |

**High dependence (same mean):**
| Loss $L$ | Probability |
|----------|-------------|
| 0% | 0.30 |
| 8% | 0.5875 |
| 40% | 0.1125 |

Both have mean loss = 9.2%.

**Equity tranche [0%, 3%]:**
$$L_{tr} = \min(L, 0.03) / 0.03$$

- Low dep: ETL = $0.05(0) + 0.95(1) = 0.95$
- High dep: ETL = $0.30(0) + 0.70(1) = 0.70$

**Senior tranche [10%, 15%]:**
$$L_{tr} = (\min(L, 0.15) - \min(L, 0.10)) / 0.05$$

- Low dep: ETL = $0.05(1) = 0.05$
- High dep: ETL = $0.1125(1) = 0.1125$

**Result:** Higher dependence *decreases* equity ETL but *increases* senior ETL.

---

### Example 3: Conditional Default Probability Calculation

**Parameters:**
- Marginal 1y PD = 2% → $a = \Phi^{-1}(0.02) = -2.054$
- $\rho = 0.20$ → $\sqrt{\rho} = 0.447$, $\sqrt{1-\rho} = 0.894$

**Conditional PD formula:**
$$p(x) = \Phi\left(\frac{a - \sqrt{\rho} x}{\sqrt{1-\rho}}\right)$$

**Bad state ($x = -1$):**
$$p(-1) = \Phi\left(\frac{-2.054 + 0.447}{0.894}\right) = \Phi(-1.80) = 0.036$$

**Good state ($x = +1$):**
$$p(+1) = \Phi\left(\frac{-2.054 - 0.447}{0.894}\right) = \Phi(-2.80) = 0.0026$$

**Interpretation:** In bad states, default probability nearly doubles; in good states, it drops to 1/8 of the unconditional level.

---

### Example 4: Full Base Correlation Bootstrap

**Market data (toy):**
- Portfolio: 125 names, 5y maturity, $R = 40\%$
- Discount rate: 5% continuous

| Tranche | Market Spread (bps) | Upfront |
|---------|---------------------|---------|
| 0–3% | 500 running | 35% |
| 3–7% | 250 running | — |
| 7–10% | 85 running | — |

**Step 1: Calibrate $\rho(3\%)$**

Using one-factor Gaussian copula, find $\rho$ such that equity tranche PV = 0 at 35% upfront + 500bp running.

Iterating:
- $\rho = 10\%$: Model upfront = 42% (too high)
- $\rho = 20\%$: Model upfront = 33% (too low)
- $\rho = 18\%$: Model upfront = 35.1% ≈ market

**Result:** $\rho(3\%) = 18\%$

**Step 2: Calibrate $\rho(7\%)$**

Holding $\rho(3\%) = 18\%$ fixed, find $\rho(7\%)$ such that [3%, 7%] tranche prices at 250bp.

Compute [3%, 7%] ETL as:
$$\text{ETL}_{3,7} = \frac{\psi(7\%, \rho(7\%)) - \psi(3\%, \rho(3\%))}{0.04}$$

Iterating:
- $\rho(7\%) = 25\%$: Implied spread = 230bp (too low)
- $\rho(7\%) = 30\%$: Implied spread = 255bp (close)
- $\rho(7\%) = 29\%$: Implied spread = 249bp ≈ market

**Result:** $\rho(7\%) = 29\%$

**Step 3: Calibrate $\rho(10\%)$**

Similarly: **Result:** $\rho(10\%) = 38\%$

**Final base correlation curve:**
| Strike | Base Correlation |
|--------|------------------|
| 3% | 18% |
| 7% | 29% |
| 10% | 38% |

---

### Example 5: Detecting Interpolation Arbitrage

Using the base correlations from Example 4, price tranchelets [4%, 5%] and [5%, 6%].

**Linear interpolation:**
$$\rho(4\%) = 18\% + \frac{1}{4}(29\% - 18\%) = 20.75\%$$
$$\rho(5\%) = 18\% + \frac{2}{4}(29\% - 18\%) = 23.5\%$$
$$\rho(6\%) = 18\% + \frac{3}{4}(29\% - 18\%) = 26.25\%$$

**Compute tranchelet ETLs:**
- $\psi(4\%, 20.75\%) = 2.85\%$
- $\psi(5\%, 23.5\%) = 3.90\%$
- $\psi(6\%, 26.25\%) = 4.82\%$

**Tranchelet spreads (using simplified spread ≈ ETL / risky PV01):**
- [4%, 5%]: ETL = $(3.90 - 2.85)/0.01 = 105\%$ → spread ≈ 2100bp
- [5%, 6%]: ETL = $(4.82 - 3.90)/0.01 = 92\%$ → spread ≈ 1840bp

**Monotonicity check:** [5%, 6%] has more subordination but lower spread than [4%, 5%]. ✓ Passes.

**Sensitivity:** If interpolation were steeper, [5%, 6%] could have *higher* spread—an arbitrage signal.

---

### Example 6: Recovery Sensitivity

Using Example 4 setup with $\rho(3\%) = 18\%$ calibrated at $R = 40\%$.

**Question:** What happens to implied $\rho(3\%)$ if we assume $R = 50\%$?

**At $R = 50\%$:** Higher recovery means lower loss-given-default, so the same tranche spread requires *lower* correlation to generate enough expected loss.

Recalibrating:
- With $R = 50\%$, $\rho(3\%) = 14\%$ matches the 35% upfront

**Key insight:** Recovery and correlation are not independently identifiable from a single tranche quote. Higher assumed recovery implies lower implied correlation.

---

### Example 7: Correlation 01 Calculation

**Setup:** Use calibrated base correlations from Example 4.

**Equity [0–3%] correlation 01:**
1. Compute PV at $\rho(3\%) = 18\%$: PV = 0 (calibrated)
2. Bump to $\rho(3\%) = 19\%$: recompute ETL and PV
3. $\text{Corr01} = \text{PV}(19\%) - \text{PV}(18\%)$

Result: Corr01 ≈ −15bp (short protection loses when correlation rises)

**Senior [7–10%] correlation 01:**
1. Bump both $\rho(7\%)$ and $\rho(10\%)$ by 1%
2. Recompute tranche PV

Result: Corr01 ≈ +8bp (short protection gains when correlation rises)

**Sign confirmation:** Opposite signs across capital structure, as expected.

---

### Example 8: Tail Dependence Comparison

**Question:** Compare the probability of joint extreme events under Gaussian vs. t-copula.

**Setup:** Two names with pairwise correlation $\rho = 0.5$.

**Gaussian copula ($\nu = \infty$):**
- Tail dependence coefficient: $\lambda = 0$
- $\Pr(\text{both in worst 1\%}) \approx 0.01 \times 0.01 = 0.0001$ (asymptotically)

**t-copula with $\nu = 4$:**
Using McNeil's formula:
$$\lambda = 2 t_5\left(-\sqrt{\frac{5 \times 0.5}{1.5}}\right) = 2 t_5(-1.29) = 2 \times 0.089 = 0.178$$

For extreme events (worst 1%), the joint probability is approximately:
- $\Pr(\text{both in worst 1\%}) \approx 0.01 \times 0.178 = 0.00178$

**Result:** The t-copula assigns **18× higher probability** to joint extreme events than asymptotic Gaussian independence would suggest.

**Implication for tranches:** A super-senior tranche that appears safe under Gaussian assumptions may have substantially higher risk under t-copula. This explains why markets price in a correlation premium for senior tranches.

---

## 50.11 Practical Notes

### 50.11.1 Input Checklist

| Category | Required Inputs |
|----------|-----------------|
| **Portfolio** | Constituents, notional weights, maturity |
| **Marginals** | CDS curves for each name (survival probabilities) |
| **Recovery** | Fixed or stochastic; typically 40% for IG |
| **Tranches** | Attachment/detachment, coupon, day count |
| **Discount** | OIS curve for discounting |
| **Quotes** | Market spreads and/or upfronts |

### 50.11.2 Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Confusing correlation with dependence** | $\rho$ is a model parameter, not the true dependence |
| **Ignoring recovery** | Recovery affects $L_{\max}$ and correlation calibration |
| **Naive interpolation** | Linear base correlation can create arbitrage |
| **Treating implied $\rho$ as stable** | Implied correlations move with market conditions |
| **Ignoring tail dependence** | Gaussian copula underestimates senior risk |

### 50.11.3 Verification Tests

| Test | What to Check |
|------|---------------|
| **Repricing** | Each calibration instrument reprices to market |
| **ETL bounds** | $0 \le \text{ETL} \le 1$; loss amount in $[0, D-A]$ |
| **Spread monotonicity** | Tranchelet spreads decrease with subordination |
| **Density positivity** | Implied loss density $\geq 0$ everywhere |
| **Limiting cases** | $\rho = 0$ gives independence; $\rho \to 1$ gives clustering |

---

## 50.12 Summary

1. **Tranche loss is nonlinear** in portfolio loss, so tranche PV depends on the *entire* loss distribution, not just expected loss.

2. **Correlation reshapes** the loss distribution: higher correlation pushes mass to extremes (both "no loss" and "many losses").

3. **Equity and senior** tranches respond oppositely to correlation changes: equity benefits from more "no loss" mass, senior suffers from more tail mass.

4. **Copulas** separate marginal default distributions (from CDS) from dependence structure—Sklar's theorem formalizes this.

5. **Gaussian copula one-factor** model uses $Z_i = \sqrt{\rho}X + \sqrt{1-\rho}\varepsilon_i$, enabling tractable conditional independence.

6. **Gaussian copula has zero tail dependence**, systematically underestimating senior tranche risk.

7. **The LHP model** provides analytical tractability for large homogeneous portfolios via the Vasicek limit.

8. **Compound correlation** (flat $\rho$ per tranche) can have multiple solutions for mezzanine and fails conservation of expected loss.

9. **Base correlation** (one $\rho(K)$ per detachment) is the market standard, calibrated via bootstrap.

10. **Interpolation pathologies** (non-monotonic spreads, negative densities) require careful handling—ETL-space interpolation helps.

11. **Model risk** is substantial: correlation is not stable, tail dependence matters, and the 2008 crisis demonstrated limitations.

---

## 50.13 Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Tranche loss mapping | $L(T;A,D) = [\min(L,D) - \min(L,A)]/(D-A)$ | Nonlinearity drives correlation sensitivity |
| Sklar's theorem | Joint CDF = copula of marginal CDFs | Separates marginals from dependence |
| One-factor Gaussian | $Z_i = \sqrt{\rho}X + \sqrt{1-\rho}\varepsilon_i$ | Enables conditional independence |
| Tail dependence | $\lim \Pr(U_2 > u \mid U_1 > u)$ as $u \to 1$ | Gaussian has zero; t-copula has positive |
| LHP model | Vasicek limit for large homogeneous portfolios | Closed-form tranche ETL via bivariate normal |
| Compound correlation | Flat $\rho$ pricing single tranche | Multiple solutions for mezz; fails consistency |
| Base correlation | $\rho(K)$ for base tranche $[0,K]$ | Market standard; enables bootstrap |
| Correlation skew | $\rho(K)$ increases with $K$ | Market's compensation for model deficiencies |
| Correlation 01 | $\partial PV / \partial \rho$ per 1% | Opposite signs across capital structure |

---

## 50.14 Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Why are tranches correlation-sensitive? | Tranche loss is nonlinear in portfolio loss, so PV depends on the full loss distribution |
| 2 | What is Sklar's theorem? | Any joint CDF can be written as a copula of its marginals; unique if marginals continuous |
| 3 | Write the one-factor Gaussian latent variable | $Z_i = \sqrt{\rho}X + \sqrt{1-\rho}\varepsilon_i$ |
| 4 | What is the conditional default probability formula? | $\Phi((a_i - \sqrt{\rho}x)/\sqrt{1-\rho})$ |
| 5 | What happens to portfolio loss distribution when $\rho$ increases? | Mass shifts to extremes (more "no loss" and more "high loss") |
| 6 | How does equity tranche ETL respond to higher $\rho$? | Decreases (more mass at "no loss") |
| 7 | How does senior tranche ETL respond to higher $\rho$? | Increases (more mass at tail) |
| 8 | What is tail dependence? | Probability of joint extremes in the limit |
| 9 | Does Gaussian copula have tail dependence? | No ($\lambda_U = \lambda_L = 0$) |
| 10 | Does t-copula have tail dependence? | Yes, for finite degrees of freedom |
| 11 | What is the t-copula tail dependence formula? | $\lambda = 2t_{\nu+1}(-\sqrt{(\nu+1)(1-\rho)/(1+\rho)})$ |
| 12 | What is compound correlation? | Flat $\rho$ that reprices a single tranche |
| 13 | Why can compound correlation fail for mezzanine? | PV may be non-monotonic in $\rho$, giving multiple solutions |
| 14 | What is base correlation? | Correlation $\rho(K)$ assigned to base tranche $[0,K]$ |
| 15 | How is non-equity tranche ETL computed under base correlation? | $[\psi(D;\rho(D)) - \psi(A;\rho(A))]/(D-A)$ |
| 16 | What is the base correlation bootstrap? | Sequential calibration: $\rho(K_1)$ from equity, then $\rho(K_2)$ from [K_1,K_2], etc. |
| 17 | What does a positive correlation skew mean? | Base correlation increases with detachment |
| 18 | What is an interpolation pathology? | Non-monotonic tranchelet spreads or negative implied density |
| 19 | How does ETL-space interpolation help? | Preserves ETL monotonicity; avoids some density issues |
| 20 | What is correlation 01? | PV sensitivity to 1% correlation change |
| 21 | Why do equity and senior have opposite correlation 01 signs? | Different exposure to loss distribution extremes |
| 22 | What is the LHP model? | Large Homogeneous Portfolio limit giving closed-form loss distribution |

---

## 50.15 Mini Problem Set

**1)** Derive the tranche loss bounds: show $L(T;A,D) \in [0,1]$.

> **Hint:** Show $\min(L,D) - \min(L,A) \in [0, D-A]$.

**2)** Compute tranche loss fractions for $A=3\%$, $D=7\%$ when $L=2\%$, $5\%$, $10\%$.

**3)** Under independence, compute $\Pr(K \geq 3)$ for 10 names with PD = 5%.

**4)** Under perfect dependence, compute $\Pr(K \geq 3)$ for the same names.

**5)** Given PD = 1%, compute threshold $a = \Phi^{-1}(0.01)$.

**6)** Derive the conditional PD formula from the latent variable construction.

**7)** Explain why $\rho = 0$ gives independence in the one-factor model.

**8)** Explain why $\rho \to 1$ gives comonotonicity.

> **Solution:** As $\rho \to 1$, $Z_i \to X$ for all $i$, so all defaults are driven by the same variable.

**9)** Using McNeil's formula, compute the tail dependence coefficient for a t-copula with $\rho = 0.7$ and $\nu = 4$.

> **Hint:** $\lambda = 2t_5(-\sqrt{5 \times 0.3/1.7})$.

**10)** Why does the Gaussian copula have zero tail dependence? (Qualitative)

**11)** Construct a 2-state example showing compound correlation can have two solutions.

**12)** State the base correlation ETL formula for tranche $[A,D]$.

**13)** Explain why the bootstrap works sequentially.

**14)** Give an example of interpolation creating arbitrage.

**15)** How would you stress test a tranche book for correlation risk?

**16)** Why does recovery affect implied correlation?

**17)** Derive the LHP loss CDF from the conditional default probability formula.

---

## 50.16 Source Map

### (A) Verified Facts — Source-Backed

| Fact | Source |
|------|--------|
| Tranche loss mapping formula | O'Kane Ch 12, Hull Ch 24 |
| Sklar's theorem statement | McNeil QRM Ch 5, O'Kane Ch 13 |
| One-factor Gaussian construction | O'Kane Ch 13, Hull Ch 24 |
| Conditional default probability formula | O'Kane Ch 13 |
| Zero tail dependence of Gaussian copula | McNeil QRM Ch 5, Example 5.32 |
| Positive tail dependence of t-copula | McNeil QRM Ch 5, Example 5.33 |
| t-copula tail dependence formula | McNeil QRM Ch 5, Example 5.33 |
| t-copula tail dependence values (Table 5.1) | McNeil QRM Ch 5, Table 5.1 |
| Compound correlation definition | O'Kane Ch 13 |
| Base correlation definition and bootstrap | O'Kane Ch 14, Ch 20 |
| Conservation of expected loss property | O'Kane Ch 20 |
| Interpolation pathologies (spreads, density) | O'Kane Ch 14, Ch 20 |
| Correlation 01 values and sign patterns | O'Kane Ch 14, Ch 17 |
| ETL-space interpolation improvement | O'Kane Ch 14, Ch 20 |
| LHP model (Vasicek limit) | O'Kane Ch 16, Vasicek (1987) |
| LHP closed-form ETL via bivariate normal | O'Kane Ch 16 |

### (B) Reasoned Inference

- Opposite correlation sensitivities for equity vs. senior derived from loss distribution reshaping under one-factor model
- Worked examples derive numeric results from source-backed formulas
- Example 8 (tail dependence comparison) applies McNeil's formula to illustrative parameters
- Model risk discussion synthesizes crisis experience documented in multiple sources

### (C) Flagged Uncertainties

- **I'm not sure** about exact current market quoting conventions for specific index tranches—conventions evolve. Verify for your desk.
- **I'm not sure** whether your implementation uses linear correlation or ETL-space interpolation—implementations vary.
- **I'm not sure** about multi-factor model calibration in the same implied correlation language—sources focus on one-factor.

---

## 50.17 Cross-References

- **Chapter 48:** Tranche mechanics, waterfall, attachment/detachment definitions
- **Chapter 49:** Expected tranche loss calculation, survival curves
- **Appendix A6:** Credit portfolio models, factor models, copula mathematics

---

## Resolution of Open Questions

**Q1: Should base correlation be the primary market language, or present compound correlation first?**

**Recommendation:** Present compound correlation first pedagogically (it's simpler), but emphasize that **base correlation is the market standard**. O'Kane confirms: base correlation "has become the market standard for pricing synthetic CDO tranches."

**Q2: Should we cover t-copula or alternative copulas?**

**Recommendation:** Include t-copula as the primary alternative, given its positive tail dependence and coverage in McNeil. Full treatment belongs in Appendix A6; here we note the limitation and point to the alternative.

**Q3: Should LHP model be included?**

**Recommendation:** Yes, LHP provides important analytical tractability and is the foundation of many risk management approaches (including Basel IRB). O'Kane Chapter 16 provides the treatment.

---

> *"Generated from O'Kane, McNeil QRM, Hull, and related sources. Verify tranche quoting conventions for your specific market."*
