# Chapter 50: Correlation and Tranche Pricing Frameworks

---

## Introduction

Why do equity and senior tranches respond in *opposite directions* to changes in "correlation"? This question lies at the heart of structured credit risk and reveals why tranche pricing cannot be reduced to simple spread-plus-leverage intuition.

Consider a portfolio of 125 investment-grade names backing a synthetic CDO. A risk manager bumps the correlation parameter from 30% to 35% and observes something counterintuitive: the equity tranche *gains* value (for a protection seller) while the super-senior tranche *loses* value. Both tranches reference the same portfolio, yet they respond with opposite signs to the same parameter change. Understanding why requires grasping how dependence reshapes the *entire distribution* of portfolio losses, not just its mean.

The Gaussian copula framework remains a common **quoting language** for tranche prices. Despite well‑known limitations (e.g., no tail dependence, calibration instabilities, base correlation interpolation pathologies), it persists because it provides a shared vocabulary for traders, a tractable calibration workflow, and a baseline that more sophisticated models can be compared against.

> **Why You Need This (For Middle Office Readers)**
>
> If you've worked in credit risk or operations, you've seen tranche positions on risk reports. The "correlation" number next to them isn't just a statistical curiosity—it's the key driver of tranche value. Understanding correlation is understanding what makes structured credit positions move, and why the 2008 crisis hit super-senior tranches that were supposed to be safe. When traders talk about being "long corr" or "short corr," they're describing the most important risk dimension of their book.

This chapter develops the conceptual and mathematical framework for correlation-sensitive tranche pricing. We begin with the fundamental question of why tranches are correlation-sensitive at all (Section 50.1), then develop copula theory from Sklar's theorem (Section 50.2). The Gaussian copula one-factor model receives detailed treatment (Section 50.3), including its critical limitation: zero tail dependence. The Large Homogeneous Portfolio (LHP) model provides analytical tractability for large portfolios (Section 50.4). We then examine the market's two correlation languages—compound correlation (Section 50.5) and base correlation (Section 50.6)—with particular attention to the pathologies that arise from base correlation interpolation (Section 50.8). Correlation trading strategies (Section 50.10) show how traders construct long and short correlation positions. Model risk and lessons from 2008 (Section 50.11) connect the framework to practical risk management, while Section 50.12 covers post-crisis evolution and bespoke tranche pricing.

**Prerequisites:** This chapter assumes familiarity with tranche mechanics and the waterfall structure (Chapter 48), expected tranche loss calculations (Chapter 49), and basic probability theory. Readers seeking deeper mathematical foundations should consult Appendix A6 on credit portfolio models.

---

## Terminology Decoder for Practitioners

| Term on Trading Desk | What It Means | Where Defined |
|---------------------|---------------|---------------|
| "Long corr" | Position profits from correlation increase | Section 50.1, 50.10 |
| "Short corr" | Position profits from correlation decrease | Section 50.1, 50.10 |
| "22 correlation on 0-3" | Base correlation of 22% for equity tranche | Section 50.6 |
| "Correlation 01" | PV change for 1% correlation move | Section 50.1.3 |
| "Equity tranche" | 0-3% (or similar) first-loss piece | Chapter 48 |
| "Super senior" | Top tranche, historically "risk-free" | Chapter 48 |
| "Skew" | Pattern of increasing base correlation with seniority | Section 50.6.6 |
| "Compound corr" | Single correlation that reprices one tranche | Section 50.5 |
| "Base corr" | Correlation for equity tranche to each strike | Section 50.6 |

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
> *   **Who loves the Magnet?**: The Equity Tranche. Why? Because in Scenario A, Equity survives. With low correlation, it’s harder for equity to have a “clean run” with no defaults; with high correlation, the probability of very low loss can increase.
> *   **Who hates the Magnet?**: The Senior Tranche. Why? Because Scenario B is the *only* way Senior loses money.

**The critical insight:** Changing correlation doesn't change the *expected* portfolio loss (which depends only on marginal default probabilities), but it dramatically changes the *shape* of the loss distribution.

### 50.1.3 Opposite Sensitivities Across the Capital Structure

This distributional reshaping affects tranches differently based on their position in the capital structure:

**Equity tranche (low attachment):** The equity tranche is concerned with whether *any* losses occur. When correlation increases, more probability mass shifts to the "no loss" scenario (everyone survives in good states). This *decreases* expected equity tranche loss.

**Senior tranche (high attachment):** The senior tranche only suffers when losses exceed its attachment point—a tail event. When correlation increases, the probability of extreme joint defaults rises. This *increases* expected senior tranche loss.

**Joint Default Intuition Table:**

| Correlation | Equity Tranche | Senior Tranche | Intuition |
|-------------|---------------|----------------|-----------|
| $\rho \to 0$ | Bad (idiosyncratic defaults eat carry) | Good (diversification works) | Independent defaults |
| $\rho \to 1$ | Good (all-or-nothing; if you survive, you survive) | Bad (fat tail, Armageddon risk) | Clustered defaults |

Practitioners often summarize this sensitivity with a “correlation 01” (Corr01): the change in tranche PV for a 1% increase in the quoted correlation parameter. The **sign pattern** is the key takeaway; the magnitude is model‑ and calibration‑dependent.

| Tranche | Corr01 sign (short protection) | Intuition |
|---------|-------------------------------|-----------|
| Equity (low attachment) | Typically **negative** | More probability of low-loss outcomes |
| Mezzanine | Can flip sign | Depends on where attachment sits vs mean/tail |
| Senior / super-senior | Typically **positive** | More tail mass where seniors get hit |

The sign flip between equity and senior is not an anomaly—it's the fundamental signature of tranche correlation sensitivity.

> **Desk Reality: What "Long Correlation" Means**
>
> When traders say "I'm long correlation," they mean they benefit if correlation increases—typically via selling equity protection or buying senior protection. A short protection position on the equity tranche is **long correlation**: the position gains when correlation rises because equity ETL decreases.
>
> Conversely, "short correlation" means benefiting from correlation decreases—typically via buying equity protection or selling senior protection.
>
> This language is the *lingua franca* of correlation desks. If you don't understand it, you can't participate in the conversation.

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

> **Practitioner Translation:** A copula separates "how likely each name defaults" (marginals) from "how likely they default together" (dependence structure). The Gaussian copula says: transform each name's default probability to a standard normal, then model joint behavior as multivariate normal.

### 50.2.2 Sklar's Theorem: The Mathematical Foundation

**Theorem (Sklar, 1959):** For any joint distribution function $H$ with marginals $F_1, \ldots, F_N$, there exists a copula $C$ such that:

$$H(x_1, \ldots, x_N) = C(F_1(x_1), \ldots, F_N(x_N))$$

If the marginals are continuous, $C$ is unique.

**Intuition:** Think of $C$ as a "coupling device." It takes $N$ uniform random variables (the probability-transformed marginals) and produces their joint distribution. The copula captures *how* the marginals are coupled—whether they tend to move together, opposite, or independently—while remaining agnostic about the marginals themselves.

McNeil, Frey, and Embrechts emphasize this core point: copulas let you study **dependence** separately from the marginals, in a way that is invariant to monotone re-parameterizations of the marginals.

### 50.2.3 The David Li Moment: Historical Context

David X. Li’s 2000 paper popularized the copula approach to default dependence in credit portfolios. The appeal is straightforward: calibrate **marginal** default behavior from CDS curves, then choose a **dependence** structure to price multi‑name payoffs such as CDO tranches.

The post‑crisis lesson is equally straightforward: tractable calibration is not the same thing as accurate tail risk. Treat “implied correlation” as a quoting convention and stress the dependence assumptions.

### 50.2.4 Common Copulas

**Independence copula:**
$$C_{\perp}(u_1, \ldots, u_N) = \prod_{i=1}^N u_i$$

No dependence—knowing one default tells you nothing about others.

**Gaussian copula:**
$$C_{\Phi}(u_1, \ldots, u_N; \Sigma) = \Phi_N(\Phi^{-1}(u_1), \ldots, \Phi^{-1}(u_N); \Sigma)$$

where $\Phi_N(\cdot; \Sigma)$ is the $N$-dimensional standard normal CDF with correlation matrix $\Sigma$.

**Student's t-copula:**
$$C_t(u_1, \ldots, u_N; \Sigma, \nu) = t_N(t_\nu^{-1}(u_1), \ldots, t_\nu^{-1}(u_N); \Sigma, \nu)$$

where $t_\nu^{-1}$ is the inverse t-distribution CDF and $\nu$ is degrees of freedom.

### 50.2.5 Why Copulas Fit Credit Portfolio Modeling

For synthetic CDOs, the copula approach is natural:

1. **Marginals are market-observable:** Single-name CDS curves give us $Q_i(t) = \Pr(\tau_i > t)$ for each name.

2. **Joint behavior is the unknown:** How defaults cluster is what we're trying to model/calibrate.

3. **Separation enables calibration:** We can calibrate marginals to single-name CDS, then calibrate the copula to tranche prices.

In other words: the copula lets you change the **dependence structure** (how defaults co‑move) without changing the **marginal** default distributions implied by single‑name CDS curves.

> **Limitation Awareness: What the Model Can't Do**
>
> The Gaussian copula has zero tail dependence (see Section 50.3.5). This means:
> - It cannot generate the kind of tail‑dependent clustering observed in many stress episodes
> - Joint extreme events can be understated relative to models with positive tail dependence
> - Senior tranche tail risk can therefore look “too small” under Gaussian assumptions
>
> Treat the model as a **quoting convention** and a starting point — then stress test what happens when dependence becomes more tail‑heavy.

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

### 50.3.4 Worked Example: Explicit Conditioning

**Setup:** Consider a homogeneous portfolio with:
- Marginal 1-year PD = 2% for all names
- Correlation $\rho = 0.25$
- 100 names with 40% recovery

**Step 1: Compute the default threshold**
$$a = \Phi^{-1}(0.02) = -2.054$$

**Step 2: Compute conditional default probabilities for different factor realizations**

| Factor $X$ | Market State | Conditional PD $p(X)$ | Interpretation |
|------------|--------------|----------------------|----------------|
| $+2$ | Very good | $\Phi\left(\frac{-2.054 - 0.5 \times 2}{0.866}\right) = \Phi(-3.53) = 0.02\%$ | Almost no defaults |
| $+1$ | Good | $\Phi\left(\frac{-2.054 - 0.5}{0.866}\right) = \Phi(-2.95) = 0.16\%$ | Very few defaults |
| $0$ | Normal | $\Phi\left(\frac{-2.054}{0.866}\right) = \Phi(-2.37) = 0.89\%$ | Below average |
| $-1$ | Bad | $\Phi\left(\frac{-2.054 + 0.5}{0.866}\right) = \Phi(-1.80) = 3.6\%$ | Elevated defaults |
| $-2$ | Very bad | $\Phi\left(\frac{-2.054 + 1.0}{0.866}\right) = \Phi(-1.22) = 11.1\%$ | Crisis-level defaults |

**Step 3: Compute conditional expected portfolio loss**

Expected portfolio loss conditional on $X$: $\mathbb{E}[L \mid X] = (1-R) \times p(X) = 0.60 \times p(X)$

| Factor $X$ | Conditional PD | Conditional Portfolio Loss |
|------------|----------------|---------------------------|
| $+2$ | 0.02% | 0.01% |
| $0$ | 0.89% | 0.53% |
| $-2$ | 11.1% | 6.7% |

**Step 4: Tranche impact**

For a 0-3% equity tranche in the $X = -2$ scenario:
- Portfolio loss of 6.7% exceeds 3% detachment
- Equity tranche is **wiped out** (100% loss)

For the same tranche in the $X = 0$ scenario:
- Portfolio loss of 0.53% is below 3% detachment
- Equity tranche loss = $\min(0.53\%, 3\%) / 3\% = 17.7\%$

**Key insight:** Conditioning creates "default clustering." In bad states, conditional PD jumps from 2% to 11%—a 5× increase. This is how correlation drives tail risk.

### 50.3.5 Limiting Cases and Sanity Checks

**$\rho = 0$ (independence):**
$$Z_i = \varepsilon_i, \quad p_i(T \mid X = x) = \Phi(a_i(T)) = \text{PD}_i(T)$$
Conditional equals marginal; defaults are independent.

**$\rho \to 1$ (comonotonicity):**
$$Z_i \approx X$$
All latent variables collapse to the single factor. Defaults become perfectly dependent—either everyone survives or everyone defaults together.

**Extreme factor realizations:**
- $X \to -\infty$: $p_i(T \mid X) \to 1$ (all default)
- $X \to +\infty$: $p_i(T \mid X) \to 0$ (none default)

### 50.3.6 Tail Dependence: The Critical Limitation

**Definition:** The upper and lower tail dependence coefficients measure the probability of joint extremes:

$$\lambda_U = \lim_{u \to 1^-} \Pr(U_2 > u \mid U_1 > u)$$
$$\lambda_L = \lim_{u \to 1^-} \Pr(U_2 \le 1-u \mid U_1 \le 1-u)$$

**The Gaussian copula has zero tail dependence:** $\lambda_U = \lambda_L = 0$ for all $\rho < 1$.

Interpretation (as emphasized in QRM): in the Gaussian case, extremes are **asymptotically independent** — seeing one name in a very extreme state conveys relatively little about another name also being extreme, unless $\rho$ is literally 1.

**Why this matters for senior tranches:** Senior tranches are tail-driven instruments. They only suffer losses in extreme scenarios where many names default together. A model with no tail dependence may systematically underestimate senior tranche risk.

**The t-copula alternative:** McNeil et al. (QRM Example 5.33) derive the tail dependence coefficient for the t-copula with correlation $\rho$ and $\nu$ degrees of freedom:

$$\boxed{\lambda_U = \lambda_L = 2 t_{\nu+1}\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)}$$

where $t_{\nu+1}(\cdot)$ is the CDF of the Student's t-distribution with $\nu+1$ degrees of freedom.

**Key observations:**
- As $\nu \to \infty$, the t-copula converges to Gaussian and tail dependence vanishes
- Lower $\nu$ (fatter tails) implies more tail dependence
- Even moderate correlation ($\rho = 0.5$) with fat tails ($\nu = 4$) gives $\lambda = 0.18$—far from zero

**Implication:** The correlation skew observed in market base correlations (Section 50.6.6) can be interpreted as the market compensating for the Gaussian copula's lack of tail dependence. Senior tranches require higher base correlation to generate sufficient ETL, implicitly pricing in tail dependence the model cannot capture.

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

### 50.4.3 LHP Formula Derivation: Step-by-Step

O'Kane (Chapter 16) provides the derivation of the analytical ETL formula. Here we show each step:

**Step 1: Define the conditional portfolio loss**

Given factor $X = x$, each name defaults independently with probability:
$$p(x) = \Phi\left(\frac{a - \sqrt{\rho}x}{\sqrt{1-\rho}}\right)$$

where $a = \Phi^{-1}(p)$ is the default threshold.

**Step 2: Apply LLN for large portfolios**

For large $N$, portfolio loss given $X = x$ converges to:
$$L \mid X = x \to (1-R) \cdot p(x)$$

**Step 3: Find the critical factor value for a given loss level**

The portfolio loss equals $\ell$ when:
$$(1-R) \cdot \Phi\left(\frac{a - \sqrt{\rho}x}{\sqrt{1-\rho}}\right) = \ell$$

Solving:
$$x^*(\ell) = \frac{a - \sqrt{1-\rho}\Phi^{-1}(\ell/(1-R))}{\sqrt{\rho}}$$

**Step 4: Derive the loss CDF**

Since $L$ is monotonically decreasing in $X$ (bad factor = high loss):
$$\Pr(L \le \ell) = \Pr(X \ge x^*(\ell)) = 1 - \Phi(x^*(\ell)) = \Phi(-x^*(\ell))$$

Substituting $x^*$:
$$F_L(\ell) = \Phi\left(\frac{\sqrt{1-\rho}\Phi^{-1}(\ell/(1-R)) - a}{\sqrt{\rho}}\right)$$

**Step 5: Analytical ETL for base tranches**

For a base tranche $[0, K]$, the expected tranche loss is:
$$\psi(T, K) = \mathbb{E}[\min(L, K)] = \int_0^K (1 - F_L(\ell)) d\ell$$

O'Kane (Chapter 16) shows this reduces to a bivariate normal integral:

$$\boxed{\psi(T, K) = (1-R) \cdot \Phi_2\left(\Phi^{-1}(p), -\Phi^{-1}\left(\frac{K}{1-R}\right); -\sqrt{\rho}\right)}$$

where $\Phi_2(a, b; \rho)$ is the bivariate standard normal CDF with correlation $\rho$.

### 50.4.4 Sanity Checks on LHP

**When $\rho = 0$ (independence):**
- Portfolio loss concentrates at $\mathbb{E}[L] = (1-R) \cdot p$ by LLN
- Variance collapses to zero
- LHP becomes deterministic at the expected loss

**When $\rho \to 1$ (perfect correlation):**
- All names default together or survive together
- Portfolio loss is $(1-R)$ with probability $p$, and $0$ with probability $1-p$
- Distribution is binary (two-point)

**Boundary behavior matches intuition:** The model correctly captures the limits of diversification ($\rho = 0$) and systemic risk ($\rho = 1$).

### 50.4.5 When LHP Applies

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

### 50.5.3 Worked Example: Two Valid Compound Correlations

Consider a 3-7% mezzanine tranche on CDX with market spread of 250bp.

**Model calculation shows:**
- At $\rho = 8\%$: Model spread = 250bp ✓
- At $\rho = 75\%$: Model spread = 250bp ✓

**Two solutions exist because:**
1. At low $\rho$: Loss distribution is concentrated; moderate losses hit mezzanine
2. At high $\rho$: Distribution is bimodal; either no loss (equity absorbs) or catastrophic loss (mezzanine wiped out)

**Why this is problematic:** The two correlations imply *completely different* loss distributions and *completely different* hedges. Which one is "right"?

### 50.5.4 Failure of Conservation of Expected Loss

Consider a portfolio with expected loss $E = \mathbb{E}[L(T)]$. By linearity:

$$\sum_{k} w_k \cdot \mathbb{E}[L(T; A_k, D_k)] = E$$

where the sum is over contiguous tranches covering the capital structure and $w_k = D_k - A_k$.

**The problem:** If we use *different* compound correlations $\rho_k$ for each tranche (as implied by market prices), we're using *different models* for each tranche. The expected losses computed under these different models will not sum to $E$.

Practically, this violates a basic consistency requirement: tranche expected losses (weighted by width) should add up to portfolio expected loss when tranches span the capital structure. Using different compound correlations tranche‑by‑tranche breaks that add‑up property.

This inconsistency motivated the development of base correlation as a more coherent quoting framework.

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

### 50.6.4 Why Base Correlation Won

> **Desk Reality: The Base Correlation Victory**
>
> Compound correlation seemed natural—one $\rho$ per tranche. But it had fatal flaws:
>
> 1. **Multiple solutions for mezzanine tranches**: Which $\rho$ is "right"?
> 2. **Non-monotonic behavior**: Higher $\rho$ doesn't always mean higher price
> 3. **No interpolation method**: You can't price a 4-5% tranche from 0-3% and 3-7%
>
> Base correlation solved all three:
> - **More stable than compound corr**: base tranche PV is typically monotonic in $\rho$, so each $K$ has a well-defined implied correlation
> - **Quotable**: you can talk about “correlation at strike $K$” just like “implied vol at strike”
> - **Interpolatable (with caveats)**: you can draw a curve through calibration points (and then deal with interpolation pathologies in Section 50.8)
>
> In practice, base correlation became a widely used market standard for quoting synthetic tranche prices.

### 50.6.5 Conservation of Expected Loss

A key property of base correlation is that it **preserves conservation of expected loss** within each base tranche calculation. O'Kane emphasizes this: for the base tranche $[0, K]$, we use a single correlation $\rho(K)$, so the model is internally consistent for that tranche.

However, when computing ETL for a mezzanine tranche $[A, D]$ using two different base correlations $\rho(A)$ and $\rho(D)$, we are effectively mixing two different models. This means:

- The *base tranche* ETLs $\psi(K; \rho(K))$ are each computed consistently
- The *difference* used for mezzanine tranches is a model approximation
- Conservation holds for the full capital structure only if we use a consistent correlation for the whole structure

**Practical implication:** Base correlation is a quoting convention, not a no-arbitrage model. It provides a common language but does not eliminate model risk.

### 50.6.6 The Base Correlation Skew

Market quotes often show a **base correlation skew**: implied base correlation tends to rise with detachment (though the curve can have kinks and even local inversions depending on the day and the interpolation method).

Hull (Table 25.8) provides one illustrative iTraxx Europe snapshot (January 31, 2007):

| Tranche | 0-3% | 0-6% | 0-9% | 0-12% | 0-22% |
|---------|------|------|------|-------|-------|
| Base Correlation | 17.7% | 28.4% | 36.5% | 43.2% | 60.5% |

**Interpretation (one way to think about it):** senior tranche pricing is driven by tail scenarios. A rising base-correlation curve is a market way of expressing “more dependence in the tail” within a Gaussian-copula quoting language.

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

Equity tranches are often quoted with an upfront plus a fixed running coupon (e.g., 500 bp in some documented index tranche quotes), while more senior tranches are often quoted as running spread only. Always confirm current conventions for the specific index/series and clearing/settlement setup.

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

Hull notes a key no‑arbitrage shape restriction: expected loss as a function of strike should increase at a **decreasing** rate (a concavity condition). Interpolating in base‑correlation space and then converting back to ETL can violate this shape condition.

**Remaining issues:** ETL-space interpolation can still produce:
- Non-smooth implied densities
- Arbitrage in dynamic hedging scenarios

### 50.8.6 Extrapolation Below 3%

The lowest quoted strike is typically 3% (the equity detachment). Pricing thin equity tranches (0–1%, 1–2%, etc.) requires extrapolation.

**The problem:** Base correlation behavior below 3% is essentially unconstrained by market data. Small changes in extrapolation assumptions can dramatically affect thin tranchelet prices.

O'Kane notes that the 0–3% tranche's implied correlation already reflects the market's view of extreme losses, and further decomposition is speculative.

> **Practitioner Workarounds**
>
> When the interpolation produces absurd results:
> 1. **Density-aware interpolation**: Use PCHIP or other shape-preserving splines
> 2. **Arbitrage constraints**: Enforce monotonicity during calibration
> 3. **Recognize the signal**: "When the curve doesn't make sense, the market is telling you something"

---

## 50.9 Alternative Dependence Models

### 50.9.1 Why Gaussian Persists Despite Known Flaws

> **Practitioner Note: The Persistence of "Wrong" Models**
>
> Despite known flaws, Gaussian copula remains the quoting convention because:
>
> 1. **Everyone speaks the same language**: Base correlation is the lingua franca
> 2. **Alternative models don't necessarily hedge better**: You're still exposed to correlation moves
> 3. **Market prices already reflect model limitations**: The skew prices in tail dependence implicitly
> 4. **"If everyone uses the wrong model consistently, it's still useful for relative value"**
>
> This parallels Black-Scholes in equity options: wrong for large moves, but still the quoting convention.

### 50.9.2 The t-Copula: Adding Tail Dependence

Hull notes that the one‑factor Gaussian copula is only one possible dependence model; many alternatives have been proposed, including the Student‑t copula.

**t-Copula calibration guidance:**

1. **Choose degrees of freedom $\nu$:**
   - Lower $\nu$ = fatter tails = more tail dependence
   - Small $\nu$ (e.g., 4) is often used as a “stressy” example; $\nu \to \infty$ recovers Gaussian

2. **Tail dependence coefficient:**
   $$\lambda = 2 t_{\nu+1}\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)$$

3. **When t-copula adds value:**
   - Senior tranche pricing (tail-sensitive)
   - Stress testing (crisis scenarios)
   - When Gaussian consistently misprices senior tranches

### 50.9.3 Random Recovery and Factor Loadings

Hull discusses extensions where correlation (or factor loadings) can be state‑dependent and recovery can be negatively related to default rates (ideas associated with Andersen and Sidenius), reflecting that “bad states” tend to feature both higher clustering and lower recoveries.

This addresses two realities:
1. **Correlation is state-dependent**: Higher in bad states
2. **Recovery is state-dependent**: Lower when default rates are high

### 50.9.4 Decision Tree for Model Choice

| Situation | Model Choice | Rationale |
|-----------|-------------|-----------|
| Quoting/trading | Gaussian + base correlation | Market convention |
| Risk management | t-copula or scenario | Capture tail risk |
| Stress testing | Multiple models | Model uncertainty |
| Exotic/bespoke pricing | Bespoke calibration | Match hedge instruments |
| Regulatory capital | Follow Basel specifications | Compliance |

---

## 50.10 Correlation Trading Strategies

### 50.10.1 Long Correlation Position

**Construction:** Sell equity protection + Buy senior protection (delta-neutral on spread)

**Mechanics:**
- Receive premium on equity (carry positive)
- Pay premium on senior (carry negative)
- Net carry depends on relative spreads

**P&L dynamics:**
- Correlation rises → Equity ETL falls → Equity position gains
- Correlation rises → Senior ETL rises → Senior position gains (you're long protection)
- **Both legs profit from correlation increase**

**Risks:**
- Correlation drops (both legs lose)
- Early defaults hit equity before correlation can rise
- Spread moves without correlation (basis risk)

Intuition: being short equity protection is often **long correlation** because increasing correlation can increase the probability of very low portfolio loss outcomes (helping equity).

### 50.10.2 Short Correlation Position

**Construction:** Buy equity protection + Sell senior protection (delta-neutral on spread)

**Mechanics:**
- Pay premium on equity
- Receive premium on senior
- Net carry typically positive (equity expensive relative to senior)

**P&L dynamics:**
- Correlation drops → Equity ETL rises → Equity position gains (you're long protection)
- Correlation drops → Senior ETL falls → Senior position gains
- **Both legs profit from correlation decrease**

**Risks:**
- Correlation spikes in crisis (2008 scenario)
- Senior tranche losses that were priced as very remote become much more plausible
- Jump-to-default on concentrated names

### 50.10.3 Delta-Neutral Correlation Trades

To isolate correlation exposure from spread exposure, traders construct **delta-neutral** positions:

**Step 1:** Calculate tranche deltas (spread sensitivity to index)
- Equity delta can be an order of magnitude larger than the index
- Senior delta is often much smaller on a per‑notional basis

**Step 2:** Size positions to offset index spread exposure
- For every $1 of equity sold, buy $k$ of senior (where $k$ = equity delta / senior delta)

**Step 3:** Monitor and rebalance
- Deltas change as spreads move
- Rebalancing required to maintain neutrality

### 50.10.4 The Carry-Correlation Tradeoff

| Tranche | Carry (Short Protection) | Correlation Position |
|---------|-------------------------|---------------------|
| Equity | High positive | Long correlation |
| Mezzanine | Moderate positive | Mixed (depends on strikes) |
| Senior | Low/negative | Short correlation |
| Super-Senior | Minimal | Strong short correlation |

**The fundamental tension:** High-carry positions (equity) are long correlation and suffer in crises. Low-carry positions (senior) are short correlation and suffer in benign environments.

### 50.10.5 How Traders Quote Correlation

> **Desk Reality: Correlation Quoting Conventions**
>
> **Quote format:** "22 correlation on 0-3" means implied base correlation of 22% for equity tranche
>
> **Bid/offer:** Quoted in correlation points, not spread
> - "21/23 on 0-3" means bid at 21% corr, offer at 23% corr
> - Lower correlation = higher equity spread (seller's market)
>
> **Correlation moves vs. spread moves:**
> - Correlation moves: Structural change in loss distribution
> - Spread moves: Change in expected loss (all tranches affected similarly)
> - Often move together, but can diverge

---

## 50.11 Model Risk and Lessons from 2008

### 50.11.1 The "Formula That Killed Wall Street"

The Gaussian copula's role in the 2007-2008 crisis has been extensively documented. Key failures:

1. **Underestimation of tail risk:** Zero tail dependence meant the model systematically underestimated the probability of widespread defaults

2. **Correlation instability:** Implied correlations proved highly unstable during the crisis, with base correlation skews steepening dramatically

3. **False precision:** The single-parameter model gave an illusion of precision while masking fundamental uncertainty about joint default behavior

4. **Calibration to benign conditions:** Models calibrated to pre-crisis data failed catastrophically when correlations spiked

### 50.11.2 What Failed in 2008

**Super-senior tranches:** Tranches rated AAA and considered "risk-free" suffered catastrophic losses. The probability assigned by Gaussian copula models was essentially zero.

**Correlation spiked beyond model range:** Implied correlations moved from 30-40% to 80%+ for senior tranches. Models calibrated at lower correlations gave nonsensical results.

**Tail dependence absent from Gaussian copula:** Joint extreme events happened with much higher probability than the model predicted.

**Mark-to-market losses on "hedged" positions:** Traders with delta-neutral correlation books found that their hedges failed as correlations and spreads moved together in unprecedented ways.

### 50.11.3 Model Risk Dimensions

| Risk Dimension | Description |
|----------------|-------------|
| **Copula choice** | Gaussian vs. t-copula vs. other; tail dependence implications |
| **Correlation stability** | Implied correlation is not constant; depends on market conditions |
| **Recovery assumptions** | Fixed vs. stochastic recovery; affects correlation calibration |
| **Marginal curve risk** | Single-name curves may be inconsistent with tranche quotes |
| **Interpolation arbitrage** | Non-standard strikes may have negative spreads or density violations |

### 50.11.4 Stress Testing Framework

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

### 50.11.5 Practical Hedging Implications

The correlation smile implies that **delta hedging with single-names is incomplete**. A perfectly delta-hedged equity tranche still has:

- **Correlation exposure:** PV changes when implied correlation moves
- **Gamma exposure:** Non-linear response to spread changes
- **Gap risk:** Discrete defaults cause discontinuous PV jumps

Traders often hedge correlation exposure by trading tranches against each other (equity vs. mezzanine), but this introduces **basis risk** if the correlation smile shifts.

In principle, the only “perfect” correlation hedge for a tranche is the **same tranche** (same strikes, maturity, and reference portfolio) in the opposite direction. In practice that hedge is often unavailable or illiquid, so desks use imperfect cross‑tranche hedges and manage the residual model/basis risk.

---

## 50.12 Post-Crisis Evolution and Bespoke Pricing

### 50.12.1 Regulatory Response

Post‑crisis, correlation trading and bespoke tranches faced increased scrutiny. Bank capital, risk governance, and model validation expectations tightened, and liquidity shifted toward more standardized products.

The exact regulatory capital and risk‑management requirements are jurisdiction‑specific and have evolved over time; treat “regulatory impacts” here as qualitative unless you are working from a current rulebook.

### 50.12.2 Market Structure Changes

**Move to standardized tranches:**
- CDX and iTraxx indices dominate
- Standard attachment points (0-3%, 3-7%, etc.)
- Central clearing for indices

**Reduced bespoke activity:**
- Pre-crisis: Large bespoke CDO market
- Post-crisis: Mostly index tranches
- Bespoke now reserved for specific hedging needs

### 50.12.3 What Practitioners Learned

> **Practitioner Note: Hard-Won Lessons**
>
> 1. **"The model is a quotation convention, not a risk measure"**: Use base correlation for quoting; use scenario analysis for risk
>
> 2. **Scenario analysis matters more than model output**: Run multiple correlation levels, multiple recovery assumptions
>
> 3. **Tail risk cannot be diversified away**: Large portfolios don't eliminate systemic risk
>
> 4. **Liquidity risk dominates in crises**: Mark-to-market losses came faster than anyone expected
>
> 5. **Model validation is not optional**: Stress test, back test, and validate continuously

### 50.12.4 Bespoke Tranche Pricing

For portfolios that are not standard indices, practitioners use **mapping techniques** to borrow correlation from liquid index tranches.

The purpose of mapping is to choose a “standard” strike $K_s^*$ on a liquid index so that its base correlation $\rho_s(K_s^*)$ can be used as an input when pricing a bespoke tranche.

**Key mapping approaches:**

1. **No-mapping (direct):** Use index base correlation at same strike
   - Simple but ignores portfolio differences

2. **ATM (at-the-money) mapping:** Match ratio of strike to expected loss
   $$\frac{K_B}{\mathbb{E}[L_B]} = \frac{K_s}{\mathbb{E}[L_s]}$$

3. **TLP (tranche loss proportion) mapping:** Match ratio of tranche loss to portfolio loss
   - Most widely used according to O'Kane

**Example from O'Kane:** A bespoke portfolio with spreads 5× the iTraxx index would use TLP mapping to determine that a 4-8% bespoke tranche should use base correlations from approximately 1-2% index strikes (equity-like behavior).

### 50.12.5 Bespoke Risk Management

Bespoke correlation risk is often hedged by expressing the bespoke tranche’s correlation sensitivity in terms of **equivalent notionals** of liquid standard tranches (the hedge is only as good as the mapping).

**Hedge calculation:**
1. Calculate base correlation sensitivity of bespoke tranche
2. Express as equivalent notionals in standard tranches
3. Execute hedges in liquid standard tranches

**Basis risk warning:** Bespoke hedges are imperfect because:
- Different underlying portfolios
- Different correlation dynamics
- Mapping assumptions may break down in stress

---

## 50.13 Worked Examples

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

### Example 9: Delta-Neutral Correlation Trade

**Setup:** CDX 5Y, correlation at mid-levels

**Position:**
- Sell $10mm notional equity protection (0-3%)
- Buy senior protection (10-15%)

**Deltas (from O'Kane):**
- Equity delta: ~20x (highly leveraged)
- Senior delta: ~2x

**Delta-neutral sizing:**
- Equity DV01: $10mm × 20 = $200mm equivalent
- Senior notional needed: $200mm / 2 = $100mm

**Resulting position:**
- Sell $10mm 0-3% protection
- Buy $100mm 10-15% protection
- Spread-neutral but long correlation

**P&L for +5% correlation move:**
- Equity: +$500k (gains from lower ETL)
- Senior: +$250k (gains from higher ETL, but you're long protection)
- Net: +$750k

---

### Example 10: Crisis Scenario Analysis

**Setup:** Portfolio with 5% expected loss, $R = 40\%$

**Normal environment ($\rho = 25\%$):**

| Tranche | ETL |
|---------|-----|
| 0-3% | 98% |
| 3-7% | 35% |
| 10-15% | 2% |
| 15-30% | 0.1% |

**Crisis environment ($\rho = 60\%$):**

| Tranche | ETL |
|---------|-----|
| 0-3% | 75% |
| 3-7% | 28% |
| 10-15% | 12% |
| 15-30% | 5% |

**Key observations:**
- Equity tranche ETL *decreased* (more "no loss" scenarios)
- Super-senior ETL *increased 50×* (tail risk materialized)
- This is exactly what happened in 2008

---

## 50.14 Practical Notes

### 50.14.1 Input Checklist

| Category | Required Inputs |
|----------|-----------------|
| **Portfolio** | Constituents, notional weights, maturity |
| **Marginals** | CDS curves for each name (survival probabilities) |
| **Recovery** | Fixed or stochastic; typically 40% for IG |
| **Tranches** | Attachment/detachment, coupon, day count |
| **Discount** | OIS curve for discounting |
| **Quotes** | Market spreads and/or upfronts |

### 50.14.2 Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Confusing correlation with dependence** | $\rho$ is a model parameter, not the true dependence |
| **Ignoring recovery** | Recovery affects $L_{\max}$ and correlation calibration |
| **Naive interpolation** | Linear base correlation can create arbitrage |
| **Treating implied $\rho$ as stable** | Implied correlations move with market conditions |
| **Ignoring tail dependence** | Gaussian copula underestimates senior risk |

### 50.14.3 Verification Tests

| Test | What to Check |
|------|---------------|
| **Repricing** | Each calibration instrument reprices to market |
| **ETL bounds** | $0 \le \text{ETL} \le 1$; loss amount in $[0, D-A]$ |
| **Spread monotonicity** | Tranchelet spreads decrease with subordination |
| **Density positivity** | Implied loss density $\geq 0$ everywhere |
| **Limiting cases** | $\rho = 0$ gives independence; $\rho \to 1$ gives clustering |

---

## 50.15 Summary

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

11. **Correlation trading** involves constructing long/short correlation positions via equity vs. senior tranches.

12. **Model risk** is substantial: correlation is not stable, tail dependence matters, and the 2008 crisis demonstrated limitations.

13. **Bespoke tranche pricing** requires mapping from liquid index correlations using TLP or ATM approaches.

---

## 50.16 Key Concepts Summary

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
| Long correlation | Sell equity + buy senior (delta-neutral) | Profit if correlation increases |
| Short correlation | Buy equity + sell senior (delta-neutral) | Profit if correlation decreases |

---

## 50.17 Career Bridge: From Middle Office to Trading Desk

> **From Middle Office to Trading Desk**
>
> On a correlation desk, you'll need to:
>
> 1. **Quote correlation, not spreads**: Translate between spread and correlation fluently
>
> 2. **Understand P&L attribution**:
>    - "How much of today's P&L came from correlation moves vs. spread moves?"
>    - Use correlation 01 for attribution
>
> 3. **Recognize when the model is being asked to do too much**:
>    - Interpolation pathologies
>    - Extrapolation below 3%
>    - Bespoke pricing with thin liquidity
>
> 4. **Explain to risk management why your "hedged" position still has risk**:
>    - Delta-neutral ≠ correlation-neutral
>    - Gap risk from defaults
>    - Basis risk between hedge instruments
>
> 5. **Know the crisis history**: Why super-senior failed, what we learned

---

## 50.18 Flashcards

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
| 23 | Why is Gaussian copula still used despite known tail dependence failures? | It's the market quoting convention; everyone speaks the same language; alternatives don't necessarily hedge better |
| 24 | What is a "long correlation" position? | Sell equity protection + buy senior protection (delta-neutral); profits if correlation increases |
| 25 | What does tail dependence measure and what is it for Gaussian copula? | Probability of joint extreme events; Gaussian copula has zero tail dependence |

---

## 50.19 Mini Problem Set

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

**18)** (Trading) Given equity and senior tranche prices, construct a delta-neutral long correlation position. Calculate P&L for a 5% correlation increase.

> **Solution sketch:** Size senior notional = (equity delta / senior delta) × equity notional. P&L = Corr01_equity × 5 + Corr01_senior × 5 (both positive for long corr position).

**19)** (Crisis Scenario) A portfolio has 5% expected loss. Compare ETL for 0-3% tranche under ρ = 0.2 (normal) vs ρ = 0.6 (crisis). Explain why super-senior losses emerged in 2008.

> **Solution sketch:** At ρ = 0.6, more probability mass moves into tail scenarios. Equity ETL can decrease, while super‑senior ETL can rise sharply. In crisis regimes, dependence repricing can make previously “remote” senior losses become much more plausible.

**20)** (Model Comparison) For the same portfolio, compute 0-3% tranche value using Gaussian copula (ρ = 0.3) and t-copula (ρ = 0.3, v = 4). Discuss which is more conservative for senior tranches.

> **Solution sketch:** t-copula has positive tail dependence, so senior tranches have higher ETL. t-copula is more conservative for senior tranches. Equity tranches may have similar or slightly lower ETL under t-copula.

---

## References

- Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives* (Gaussian copula; compound/base correlation; calibration and mapping; hedging)
- McNeil, Frey, Embrechts, *Quantitative Risk Management* (copulas; tail dependence; t‑copula properties)
- John C. Hull, *Options, Futures, and Other Derivatives* (CDO / tranche modeling overview and examples)
- David X. Li (2000), “On Default Correlation: A Copula Function Approach” (copula approach in credit)

---

## 50.21 Cross-References

- **Chapter 48:** Tranche mechanics, waterfall, attachment/detachment definitions
- **Chapter 49:** Expected tranche loss calculation, survival curves
- **Chapter 51:** Tranche delta, correlation risk hedging, jump-to-default
- **Chapter 52:** Credit trading strategies bringing these concepts together
- **Appendix A6:** Credit portfolio models, factor models, copula mathematics

---
*Chapter 50 of Fixed Income: Practice and Theory*
