# Chapter 50: Correlation and Tranche Pricing Frameworks

---

## Introduction

Why do equity and senior tranches respond in *opposite directions* to changes in "correlation"? This question lies at the heart of structured credit risk and reveals why tranche pricing cannot be reduced to simple spread-plus-leverage intuition.

Consider a portfolio of 125 investment-grade names backing a synthetic CDO. A risk manager bumps the correlation parameter from 30% to 35% and observes something counterintuitive: the equity tranche *gains* value (for a protection seller) while the super-senior tranche *loses* value. Both tranches reference the same portfolio, yet they respond with opposite signs to the same parameter change. Understanding why requires grasping how dependence reshapes the *entire distribution* of portfolio losses, not just its mean.

The Gaussian copula framework remains a common **quoting language** for tranche prices. Despite well‑known limitations (e.g., no tail dependence, calibration instabilities, base correlation interpolation pathologies), it persists because it provides a shared vocabulary for traders, a tractable calibration workflow, and a baseline that more sophisticated models can be compared against.

> **Desk Reality:** In risk systems, the “correlation” shown for a tranche is usually a *model-implied quoting parameter* (e.g., a base-correlation node or a compound-correlation value) used to make tranche PV and risk measures like Corr01 consistent with market tranche quotes.
> **Common break:** Two systems can disagree on “correlation” for the same tranche because they use different calibration instruments, interpolation/extrapolation rules, and bump/rebuild methodologies.
> **What to check:** Reprice with a clearly stated methodology (compound vs. base) and compute Corr01 from two consistent reprices; sanity-check tranchelet spreads and implied loss densities for obvious arbitrage artifacts.

This chapter develops the conceptual and mathematical framework for correlation-sensitive tranche pricing. We begin with the fundamental question of why tranches are correlation-sensitive at all (Section 50.1), then develop copula theory from Sklar's theorem (Section 50.2). The Gaussian copula one-factor model receives detailed treatment (Section 50.3), including its critical limitation: zero tail dependence. The Large Homogeneous Portfolio (LHP) model provides analytical tractability for large portfolios (Section 50.4). We then examine the market's two correlation languages—compound correlation (Section 50.5) and base correlation (Section 50.6)—with particular attention to the pathologies that arise from base correlation interpolation (Section 50.8). Correlation trading strategies (Section 50.10) show how traders construct long and short correlation positions. Section 50.11 summarizes key model-risk dimensions and stress-testing habits, and Section 50.12 gives a high-level view of bespoke tranche mapping.

**Prerequisites:** [Chapter 48](chapter_48_cdo_tranche_products_product_map.md), [Chapter 49](chapter_49_tranche_core_concepts_etl_pv.md), and basic probability.

**Follow-on:** [Chapter 51](chapter_51_tranche_risk.md), [Chapter 52](chapter_52_credit_trading_strategies.md), and [Appendix A6](appendix_a6_credit_portfolio_modeling.md).

## Learning Objectives
- Explain why tranche PV depends on the *shape* of the portfolio loss distribution (not just expected loss).
- Use copulas (via Sklar’s theorem) to separate marginal default behavior from dependence assumptions.
- Write and interpret the one-factor Gaussian latent-variable model and its conditional default probability.
- Distinguish compound vs. base correlation as quoting languages, and state the core ETL decomposition used in base correlation.
- Define Corr01 with an explicit bump object, bump size, units, and sign convention; apply basic sanity checks for interpolation artifacts.

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
Throughout this chapter, losses and attachment/detachment points are expressed as **fractions of portfolio notional** (e.g., 3% ≡ 0.03). A consolidated symbol table is provided in the end-matter `## Notation`.

## 50.1 Why Tranche Valuation Is Correlation-Sensitive

### 50.1.1 The Nonlinearity at the Core

The fundamental reason tranches are correlation-sensitive lies in a simple mathematical fact: tranche loss is a *nonlinear* function of portfolio loss. Recall from Chapter 48 the tranche loss mapping:

$$\boxed{L(T; A, D) = \frac{\min(L(T), D) - \min(L(T), A)}{D - A}}$$

This piecewise-linear function has kinks at $A$ and $D$. Below $A$, the tranche suffers no loss (subordination absorbs everything). Between $A$ and $D$, losses pass through dollar-for-dollar. Above $D$, the tranche is wiped out.

**Why nonlinearity matters:** Because tranche loss is nonlinear in portfolio loss, the expected tranche loss $\mathbb{E}[L(T; A, D)]$ depends on the entire distribution of $L(T)$, not just its mean. Two portfolios with identical expected losses but different loss distributions will have different tranche expected losses.

This is Jensen's inequality at work: for a nonlinear function $f$, generally $\mathbb{E}[f(X)] \neq f(\mathbb{E}[X])$.

### 50.1.2 How Correlation Reshapes the Loss Distribution

In a one-factor Gaussian latent-variable setup, the dependence parameter controls how much portfolio loss is driven by the systematic factor $X$ versus idiosyncratic shocks:

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

Define Corr01 for the **stated position** as:

$$\boxed{\\mathrm{Corr01} := PV(\rho+0.01)-PV(\rho)}$$

- **Bump object:** specify what “$\rho$” means (flat compound correlation, a node on a base-correlation curve/surface, or another dependence parameter) and how the curve/surface is rebuilt after the bump.
- **Bump size:** an absolute +1% correlation bump (e.g., 0.30 → 0.31).
- **Units:** currency per 1% correlation.
- **Interpretation:** Corr01 > 0 means the position’s PV increases when the desk’s dependence parameter increases.

**Check (finite-difference robustness):** tranche PV can be nonlinear in the dependence parameter and base-correlation bumps can propagate non-locally through interpolation. A simple diagnostic is to compute Corr01 using both a one-sided bump and a symmetric difference,
$$
\mathrm{Corr01}_{\text{sym}} \approx \frac{PV(\rho+0.01)-PV(\rho-0.01)}{2},
$$
holding the same calibration/interpolation rules fixed. Large discrepancies flag nonlinearity or methodology instability (and should push you toward scenario shocks rather than relying on a local number).

| Tranche | Corr01 sign (short protection) | Corr01 sign (long protection) | Intuition |
|---------|-------------------------------|------------------------------|-----------|
| Equity (low attachment) | Typically **positive** | Typically **negative** | More mass at “very low loss” helps equity protection sellers |
| Mezzanine | Can flip sign | Can flip sign | Depends on strikes vs the mean/tail of the loss distribution |
| Senior / super-senior | Typically **negative** | Typically **positive** | More tail mass hurts senior protection sellers |

The sign flip between equity and senior is not an anomaly—it's the fundamental signature of tranche correlation sensitivity.

> **Desk Reality:** “Long correlation” means *PV increases when the desk’s dependence parameter increases* (under the desk’s pricing and calibration methodology).
> **Common break:** People mix position direction (long/short protection), correlation definition (compound vs. base), and bump/rebuild rules, which can flip the sign.
> **What to check:** Compute Corr01 from two reprices $PV(\rho+1\\%)-PV(\rho)$ for the stated position, holding calibration and interpolation rules fixed.

### 50.1.4 Position Matters: Long vs. Short Protection

The above sensitivities assume a **short protection** position (receiving premium, paying losses). For a **long protection** position, all signs reverse:

- Long protection equity: *short* correlation (loses when correlation rises)
- Long protection senior: *long* correlation (gains when correlation rises)

This distinction is crucial for hedging. A trader hedging equity risk with senior tranches is implicitly taking a correlation view.

### Worked Example — Why Equity and Super-Senior Can Have Opposite Corr01 Signs (Toy)

**Example Title**: Correlation-sensitivity sign check via a one-period toy PV

**Context**
- We want a concrete sanity check for the **sign** of correlation sensitivity across the capital structure.
- This is a *toy* calculation to build intuition. Real tranche PVs use multi-period premium schedules, default timing within periods, discounting, and a specific calibration (compound or base correlation).

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-15
- Settlement date: 2026-02-17 (assume T+2)
- Accrual start/end: 2026-02-17 to 2031-02-15 (toy: one accrual period)
- Payment date(s): 2031-02-15 (toy: a single premium payment and a single protection payment at maturity)

**Inputs**
- Portfolio: $N=125$ equal-weight names; horizon $T=5$ years.
- Marginal default probability to $T$: $p=2\\%$ (hypothetical).
- Recovery: $R=40\\%$ so $\mathrm{LGD}=60\\%$.
- Tranches (loss strikes are fractions of portfolio notional):
  - Equity: $[A,D]=[0\\%,3\\%]$
  - Super-senior: $[A,D]=[30\\%,100\\%]$
- Position: **short protection** on each tranche, tranche notional $N_{\text{tr}}=USD 10$mm.
- Premium: running spread $s=500$bp per year (hypothetical), paid on **surviving tranche notional**.
- Discounting: ignore for sign intuition (set $Z(0,T)\approx 1$).
- Two dependence extremes (limiting cases):
  - **Independence** ($\rho=0$): defaults are independent across names.
  - **Perfect dependence / “all-or-nothing”** ($\rho \to 1$): either everyone defaults or nobody defaults, matching the same marginal $p$.

**Outputs (What You Produce)**
- Expected tranche loss fraction (ETL): $\mathbb{E}[L(T;A,D)]$ (unitless, in $[0,1]$).
- Expected surviving fraction: $\mathbb{E}[1-L(T;A,D)]$ (unitless).
- Toy PV per USD 1 of tranche notional (short protection):
  $$
  PV \approx sT\\,\\mathbb{E}[1-L(T;A,D)]-\\mathbb{E}[L(T;A,D)].
  $$

**Step-by-step**
1. Under independence, the number of defaults $K\sim\\mathrm{Binomial}(N,p)$; portfolio loss is $L(T)=\\mathrm{LGD}\\,(K/N)$.
2. Map portfolio loss to tranche loss fraction:
   $$
   L(T;A,D)=\frac{\\min(L(T),D)-\\min(L(T),A)}{D-A}.
   $$
3. Compute ETL $=\\mathbb{E}[L(T;A,D)]$ and expected survival $=\\mathbb{E}[1-L(T;A,D)]$.
4. Plug into the toy PV formula above.

**Cashflows (table)** *(toy: expected values shown at maturity)*
| Date | Cashflow | Explanation |
|---|---|---|
| 2031-02-15 | $+sT\\,N_{\text{tr}}\\,\\mathbb{E}[1-L(T;A,D)]$ | Premium received by short-protection holder |
| 2031-02-15 | $-N_{\text{tr}}\\,\\mathbb{E}[L(T;A,D)]$ | Expected protection payment by short-protection holder |

**Results (numbers)** *(with the inputs above; independence computed under a binomial model)*
| Tranche | $\mathbb{E}[L]$ at $\rho=0$ | $\mathbb{E}[L]$ at $\rho\to 1$ | Toy PV (short prot) at $\rho=0$ | Toy PV (short prot) at $\rho\to 1$ |
|---|---:|---:|---:|---:|
| Equity $0\text{–}3\\%$ | 0.3976 | 0.0200 | $-0.2470\times N_{\text{tr}}$ | $+0.2250\times N_{\text{tr}}$ |
| Super-senior $30\text{–}100\\%$ | $\approx 0$ | 0.0086 | $+0.2500\times N_{\text{tr}}$ | $+0.2393\times N_{\text{tr}}$ |

For $N_{\text{tr}}=USD 10\text{mm}$, the equity PV increases by about $USD 4.72\text{mm}$ when moving from $\rho=0$ to $\rho\to 1$, while the super-senior PV decreases by about $USD 0.11\text{mm}$.

**P&L / Risk Interpretation**
- The **equity short-protection** position benefits when dependence increases (PV up): it is **long correlation** under the book’s sign convention.
- The **super-senior short-protection** position is hurt when dependence increases (PV down): it is **short correlation**.
- For **long protection**, all PV changes reverse.

**Sanity Checks**
- Units: $sT$ is unitless, $\\mathbb{E}[1-L]$ is unitless → premium is a fraction of notional; ETL is a fraction of notional.
- Limiting-case check: under $\rho\to 1$, outcomes become “all-or-nothing,” which pushes mass to the extremes and can help low-attachment tranches while hurting high-attachment tranches.
- Sign check: the PV change direction matches the Corr01 sign pattern table in Section 50.1.3.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you define PV from the correct side (short vs long protection)?
- Did you specify what “$\rho$” means (compound vs base) and how you rebuild after a bump?
- Are you holding marginals fixed when you change dependence (so expected portfolio loss stays the same)?
- Are your tranche strikes in **decimals** (0.03, not 3)?

---

## 50.2 Copula Theory: Linking Marginals to Joint Behavior

### 50.2.1 The Separation Problem

Credit portfolio modeling faces a fundamental challenge: we have good tools for estimating *individual* default probabilities (from CDS curves), but we need to price instruments that depend on *joint* default behavior.

Copulas provide a mathematically elegant solution by separating these two problems:
1. **Marginal distributions:** How likely is each name to default by time $t$?
2. **Dependence structure:** Given these marginals, how do defaults cluster together?

> **Practitioner Translation:** A copula separates "how likely each name defaults" (marginals) from "how likely they default together" (dependence structure). The Gaussian copula says: transform each name's default probability to a standard normal, then model joint behavior as multivariate normal.

### 50.2.2 Sklar's Theorem: The Mathematical Foundation

**Theorem (Sklar):** For any joint distribution function $H$ with marginals $F_1, \ldots, F_N$, there exists a copula $C$ such that:

$$H(x_1, \ldots, x_N) = C(F_1(x_1), \ldots, F_N(x_N))$$

If the marginals are continuous, $C$ is unique.

**Intuition:** Think of $C$ as a "coupling device." It takes $N$ uniform random variables (the probability-transformed marginals) and produces their joint distribution. The copula captures *how* the marginals are coupled—whether they tend to move together, opposite, or independently—while remaining agnostic about the marginals themselves.

A key point: copulas let you study **dependence** separately from marginals.

### 50.2.3 Historical Context: Copulas in Credit

An influential paper popularized the copula approach to default dependence in credit portfolios. The appeal is straightforward: calibrate **marginal** default behavior from CDS curves, then choose a **dependence** structure to price multi‑name payoffs such as CDO tranches.

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

1. **Marginals are market-observable:** Single-name CDS curves give us $Q_i(t) = \Pr(\tau_i \gt t)$ for each name.

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

**Equivalent notation (common in credit texts):** the same one-factor Gaussian latent-variable construction is often written as
$$\boxed{A_i=\beta_i Z+\sqrt{1-\beta_i^2}\\,\varepsilon_i}$$
with a market factor $Z\sim N(0,1)$ and a loading $\beta_i\in[0,1]$. In the flat-correlation case, $\beta_i=\sqrt{\rho}$ and $Z=X$.

**Verification:** $Z_i$ is standard normal: $\mathbb{E}[Z_i] = 0$, $\text{Var}(Z_i) = \rho + (1-\rho) = 1$.

**Pairwise correlation:** For $i \neq j$:
$$\text{Corr}(Z_i, Z_j) = \mathbb{E}[Z_i Z_j] = \rho \mathbb{E}[X^2] + 0 = \rho$$

This is the "flat correlation" assumption: all pairs have the same correlation $\rho$.

### 50.3.2 Default Thresholds from Marginal Probabilities

Given marginal default probability $PD_i(T) = 1 - Q_i(T)$ from CDS calibration, we set a threshold:

$$a_i(T) = \Phi^{-1}(PD_i(T))$$

and define default by time $T$ as:

$$\tau_i \le T \iff Z_i \le a_i(T)$$

**Why this works:** $\Pr(Z_i \le a_i(T)) = \Phi(a_i(T)) = PD_i(T)$, matching the marginal.

Equivalently, in terms of survival probability $Q_i(T)$, the threshold can be written as $a_i(T)=\Phi^{-1}(1-Q_i(T))$.

### 50.3.3 Conditional Independence: The Computational Key

Conditional on $X = x$, the $Z_i$ are independent normal random variables:

$$Z_i \mid X = x \sim N(\sqrt{\rho} x, 1 - \rho)$$

This yields the **conditional default probability**:

$$\boxed{p_i(T \mid X = x) = \Phi\left(\frac{a_i(T) - \sqrt{\rho} x}{\sqrt{1-\rho}}\right)}$$

In the common $(A_i,\beta_i,Z)$ notation above, the same object is written as
$$\boxed{p_i(T\mid Z)=\Phi\left(\frac{C_i(T)-\beta_i Z}{\sqrt{1-\beta_i^2}}\right),\qquad C_i(T)=\Phi^{-1}(1-Q_i(T)).}$$

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
| $+2$ | Very good | $\Phi\left(\frac{-2.054 - 0.5 \times 2}{0.866}\right) = \Phi(-3.53) = 0.02\\%$ | Almost no defaults |
| $+1$ | Good | $\Phi\left(\frac{-2.054 - 0.5}{0.866}\right) = \Phi(-2.95) = 0.16\\%$ | Very few defaults |
| $0$ | Normal | $\Phi\left(\frac{-2.054}{0.866}\right) = \Phi(-2.37) = 0.89\\%$ | Below average |
| $-1$ | Bad | $\Phi\left(\frac{-2.054 + 0.5}{0.866}\right) = \Phi(-1.80) = 3.6\\%$ | Elevated defaults |
| $-2$ | Very bad | $\Phi\left(\frac{-2.054 + 1.0}{0.866}\right) = \Phi(-1.22) = 11.1\\%$ | Crisis-level defaults |

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
- Equity tranche loss = $\min(0.53\\%, 3\\%) / 3\\% = 17.7\\%$

**Key insight:** Conditioning creates "default clustering." In bad states, conditional PD jumps from 2% to 11%—a 5× increase. This is how correlation drives tail risk.

### 50.3.5 Limiting Cases and Sanity Checks

**$\rho = 0$ (independence):**
$$Z_i = \varepsilon_i, \quad p_i(T \mid X = x) = \Phi(a_i(T)) = PD_i(T)$$
Conditional equals marginal; defaults are independent.

**$\rho \to 1$ (comonotonicity):**
$$Z_i \approx X$$
All latent variables collapse to the single factor. Defaults become perfectly dependent—either everyone survives or everyone defaults together.

**Extreme factor realizations:**
- $X \to -\infty$: $p_i(T \mid X) \to 1$ (all default)
- $X \to +\infty$: $p_i(T \mid X) \to 0$ (none default)

### 50.3.6 Tail Dependence: The Critical Limitation

**Definition (Upper and lower tail dependence):** Given r.v.'s $X_1$ and $X_2$ with marginal distributions $F_1$ and $F_2$,
$$
\begin{aligned}
\lambda_{\mathrm{U}} & =\lim _{u \uparrow 1} \mathbb{P}\\!\left[X_{2}\gt F_{2}^{-1}(u) \mid X_{1}\gt F_{1}^{-1}(u)\right], \\
\lambda_{\mathrm{L}} & =\lim _{u \downarrow 0} \mathbb{P}\\!\left[X_{2} \leq F_{2}^{-1}(u) \mid X_{1} \leq F_{1}^{-1}(u)\right] .
\end{aligned}
$$

Intuition: $\lambda_{\mathrm{U}}$ is a limiting conditional probability of a joint extreme in the upper tail; $\lambda_{\mathrm{L}}$ is the analogous object in the lower tail.

**The Gaussian copula has zero tail dependence:** $\lambda_{\mathrm{U}} = \lambda_{\mathrm{L}} = 0$ for all $\rho \lt 1$. (Only at $\rho=1$ do these coefficients jump to 1.)

Interpretation: in the Gaussian case, extremes are **asymptotically independent** — seeing one name in a very extreme state does not force another name to also be extreme, unless $\rho$ is literally 1.

**Check (asymptotic vs finite-tail intuition):** “zero tail dependence” is a *limit* statement. At finite but high quantiles (e.g., 99th percentile moves), a Gaussian copula with high $\rho$ can still generate strong joint stress; it just does not deliver a strictly positive limiting conditional probability as you push the quantile to 100%.

**Why this matters for senior tranches:** Senior tranches are tail-driven instruments. They only suffer losses in extreme scenarios where many names default together. A model with no tail dependence may systematically underestimate senior tranche risk.

**The t-copula alternative:** Unlike the Gaussian copula, the Student‑t copula has non-zero tail dependence. For a t-copula with correlation $\rho$ and $\nu$ degrees of freedom:

$$\boxed{\lambda_{\mathrm{U}} = \lambda_{\mathrm{L}} = 2 t_{\nu+1}\left(-\frac{\sqrt{\nu+1}\sqrt{1-\rho}}{\sqrt{1+\rho}}\right)}$$

where $t_{\nu+1}(\cdot)$ is the CDF of the Student's t-distribution with $\nu+1$ degrees of freedom.

**Key observations:**
- As $\nu \to \infty$, the t-copula converges to Gaussian and tail dependence vanishes
- Lower $\nu$ (fatter tails) implies more tail dependence
- For example, with $\rho = 0.5$ and $\nu = 4$, $\lambda \approx 0.253$—far from zero

**Implication (one interpretation):** The correlation skew observed in market base correlations (Section 50.6.6) can be read as a “tail-risk adjustment” inside a Gaussian-copula quoting language: senior tranches need more dependence to generate enough tail loss in the model.

---

## 50.4 Large Homogeneous Portfolio (LHP) Model

### 50.4.1 The Vasicek Limit

For large portfolios with identical names, the one-factor Gaussian copula admits a powerful analytical simplification known as the Large Homogeneous Portfolio (LHP) model (often called the Vasicek limit).

**Assumptions:**
- $N$ names with identical marginal default probabilities: $PD_i(T) = p$ for all $i$
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

The analytical ETL formula in the LHP limit follows from a one-factor conditional-independence construction. Here are the steps:

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

This reduces to a bivariate normal integral:

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

In practice, mezzanine tranche PV can be non-monotonic in $\rho$ over some ranges, producing multiple implied compound-correlation roots.

### 50.5.3 Worked Example: Two Valid Compound Correlations

This is an illustrative toy scenario: a mezzanine tranche can have multiple implied compound correlations because the tranche PV can be non-monotonic in $\rho$.

Consider a 3-7% mezzanine tranche on CDX quoted at a running spread of 250bp (hypothetical).

**Model calculation shows:**
- At $\rho = 8\\%$: Model spread = 250bp ✓
- At $\rho = 75\\%$: Model spread = 250bp ✓

**Two solutions exist because:**
1. At low $\rho$: Loss distribution is concentrated; moderate losses hit mezzanine
2. At high $\rho$: Distribution is bimodal; either no loss (equity absorbs) or catastrophic loss (mezzanine wiped out)

**Why this is problematic:** The two correlations imply *completely different* loss distributions and *completely different* hedges. Which one is "right"?

The exact values depend on the marginals, recovery, maturity, and premium schedule; the point is that mezzanine-tranche PV need not be monotone in $\rho$.

### 50.5.4 Failure of Conservation of Expected Loss

Consider a portfolio with expected loss $E = \mathbb{E}[L(T)]$. By linearity:

$$\sum_{k} w_k \cdot \mathbb{E}[L(T; A_k, D_k)] = E$$

where the sum is over contiguous tranches covering the capital structure and $w_k = D_k - A_k$.

**The problem:** If we use different compound correlations $\rho_k$ for each tranche (as implied by market prices), we're using different models for each tranche. The expected losses computed under these different models will not sum to $E$.

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
> *   **The Result**: A "Skew" of correlations. Often $\rho_3 \lt \rho_7 \lt \rho_{10}$.
> *   **Why?**: Senior tranches effectively "bid up" correlation (demand protection against tail risk), just like deep OTM puts bid up implied volatility (skew).

### 50.6.2 Pricing Non-Equity Tranches

Any tranche $[A, D]$ is priced as the difference of two base tranches:

$$\boxed{\mathbb{E}[L(T; A, D)] = \frac{\psi(T, D) - \psi(T, A)}{D - A}}$$

where the base-tranche ETL function is
$$
\psi(T,K):=\mathbf{E}_{\rho(K)}[\min(L(T),K)].
$$

**Note the asymmetry:** The tranche $[A, D]$ uses *two different* correlations: $\rho(A)$ for the lower bound and $\rho(D)$ for the upper bound.

### 50.6.3 The Bootstrap Algorithm

A common workflow is a sequential calibration (“bootstrap”) up the capital structure:

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

A key property of base correlation is that it is internally consistent **for each base tranche** $[0,K]$: the base-tranche ETL $\psi(T,K)$ is computed under a single parameter $\rho(K)$.

However, when computing ETL for a mezzanine tranche $[A, D]$ using two different base correlations $\rho(A)$ and $\rho(D)$, we are effectively mixing two different models. This means:

- The *base tranche* ETLs $\psi(K; \rho(K))$ are each computed consistently
- The *difference* used for mezzanine tranches is a model approximation
- Conservation holds for the full capital structure only if we use a consistent correlation for the whole structure

**Practical implication:** Base correlation is a quoting convention, not a no-arbitrage model. It provides a common language but does not eliminate model risk.

### 50.6.6 The Base Correlation Skew

Market quotes often show a **base correlation skew**: implied base correlation tends to rise with detachment (though the curve can have kinks and even local inversions depending on quotes and the interpolation method).

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

For CDX.NA.IG and iTraxx Europe, a commonly used **six-tranche** strike grid is:

- CDX.NA.IG: 0–3%, 3–7%, 7–10%, 10–15%, 15–30%, 30–100%
- iTraxx Europe: 0–3%, 3–6%, 6–9%, 9–12%, 12–22%, 22–100%

Desk labels vary (what one desk calls “senior” vs “super-senior”), so always state the **attachment/detachment** explicitly.

Index tranche quotes are commonly given as either a running spread only, or an upfront amount plus a standardized running coupon (equity tranches are often in the second format). Always confirm quoting and settlement conventions for the specific index/series and clearing/settlement setup.

### 50.7.3 Calibration Objective

For each tranche, find correlation such that:

$$PV_{\text{model}}(\text{correlation}) = PV_{\text{market}}$$

At initiation with market quotes, this typically means $PV = 0$.

---

## 50.8 Base Correlation Interpolation Pathologies

### 50.8.1 The Interpolation Problem

Standard tranches have discrete attachment points (3%, 7%, 10%, etc.). To price **bespoke tranches** or **tranchelets** at non-standard strikes, we must interpolate the base correlation curve.

The interpolation rule is not just “cosmetic”: it can change implied loss distributions and even create arbitrage-like artifacts.

> **Pitfall — Base correlation interpolation artifacts:** Interpolating base correlation directly can imply non-monotone tranchelet spreads or a negative implied loss density.
> **Why it matters:** You can generate internally inconsistent prices and misleading hedges (including **non-local** Corr01 exposures).
> **Quick check:** price 1%-wide tranchelets across strikes and confirm spreads decrease with subordination; approximate the implied loss density via finite differences of $\psi(T,K)$ and confirm it is non-negative.

### 50.8.2 Non-Monotonic Tranchelet Spreads

A fundamental consistency requirement: tranchelet spreads should decrease as subordination increases. More subordination means more protection, hence lower spread.

**The pathology:** Linear interpolation of base correlation can produce tranchelet spreads that *increase* with subordination in certain strike ranges.

### 50.8.3 Negative Tranchelet Spreads

In extreme cases, interpolation can produce **negative implied spreads** for thin tranchelets. This is economically absurd—it would imply the protection buyer pays the seller.

**Why it happens:** The base correlation formula computes tranche ETL as a difference of two base tranche ETLs. If the interpolated correlations cause the lower base tranche ETL to exceed the upper (adjusted for width), the result is negative.

### 50.8.4 Implied Loss Density Spikes

Define the base-tranche expected loss function:

$$\psi(T,K)=\mathbf{E}_{\rho(K)}[\min(L(T),K)] \qquad \text{(base tranche }[0,K]\text{)}.$$

The **implied portfolio loss density** satisfies:

$$\boxed{f(K)=-\frac{\partial^2 \psi(T,K)}{\partial K^2}}$$

For a valid probability distribution, $f(K)\ge 0$ everywhere.

**Check (finite-difference density approximation):** on a uniform strike grid with spacing $\Delta K$, a quick diagnostic is
$$
f(K_i)\ \approx\ -\frac{\psi(T,K_{i+1}) - 2\psi(T,K_i) + \psi(T,K_{i-1})}{(\Delta K)^2}.
$$
Negative values indicate an interpolation/calibration artifact (not a “negative probability”), and they often coincide with negative tranchelet spreads or unstable Corr01 hedges.

**The pathology:** Interpolation can produce:
- **Spikes:** Unrealistically high density at certain strikes
- **Negative regions:** $f(K) \lt 0$, which is mathematically impossible for a true distribution

In practice, these show up as non-monotone tranchelet spreads, unstable hedge ratios, and “bumps that propagate” far from the bumped node.

### 50.8.5 ETL-Space Interpolation

A partial remedy: instead of interpolating base *correlation*, interpolate base tranche *expected loss* (ETL) directly.

$$\psi(T, K) = \text{interpolate}(\\{\psi(T, K_i)\\})$$

Then back out the implied correlation from interpolated ETL.

**Advantages:**
- ETL is bounded in $[0, K]$ by construction
- Linear ETL interpolation preserves monotonicity of tranchelet ETL
- Avoids some (but not all) density pathologies

**Important practical nuance:** interpolation schemes can be **non-local** (especially splines). Bumping a quoted strike can change the interpolated ETL (and hence implied correlations) at other strikes, which matters for hedging and risk attribution.

**Remaining issues:** ETL-space interpolation can still produce:
- Non-smooth implied densities
- Arbitrage in dynamic hedging scenarios

### 50.8.6 Extrapolation Below 3%

The lowest quoted strike is typically 3% (the equity detachment). Pricing thin equity tranches (0–1%, 1–2%, etc.) requires extrapolation.

**The problem:** Base correlation behavior below 3% is essentially unconstrained by market data. Small changes in extrapolation assumptions can dramatically affect thin tranchelet prices.

A common practical view is that the 0–3% tranche already prices “extreme-loss” scenarios in an aggregated way; decomposing below 3% is therefore more extrapolation than inference.

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
> 3. **Market quotes reflect model limitations**: the observed skew often compensates for what a flat-$\rho$ Gaussian model misses
> 4. **"If everyone uses the wrong model consistently, it's still useful for relative value"**
>
> This parallels Black-Scholes in equity options: wrong for large moves, but still the quoting convention.

### 50.9.2 The t-Copula: Adding Tail Dependence

The one‑factor Gaussian copula is only one possible dependence model; a common alternative is the Student‑t copula, which introduces tail dependence.

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

Some extensions make correlation (or factor loadings) state‑dependent and allow recovery to co-move with the systematic factor, reflecting that “bad states” can feature both higher clustering and worse recoveries.

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
- Receive premium on equity
- Pay premium on senior
- Net carry depends on relative tranche quotes and expected losses (treat as a market input)

**P&L dynamics:**
- Correlation rises → Equity ETL falls → Equity position gains
- Correlation rises → Senior ETL rises → Senior position gains (you're long protection)
- **Both legs profit from correlation increase**

**Risks:**
- Correlation drops (both legs lose)
- Early defaults hit equity before correlation can rise
- Spread moves without correlation (basis risk)

Intuition: being short equity protection can be **long correlation** because increasing correlation can increase the probability of very low portfolio loss outcomes (helping equity).

### 50.10.2 Short Correlation Position

**Construction:** Buy equity protection + Sell senior protection (delta-neutral on spread)

**Mechanics:**
- Pay premium on equity
- Receive premium on senior
- Net carry depends on relative tranche quotes and expected losses (treat as a market input)

**P&L dynamics:**
- Correlation drops → Equity ETL rises → Equity position gains (you're long protection)
- Correlation drops → Senior ETL falls → Senior position gains
- **Both legs profit from correlation decrease**

**Risks:**
- Correlation spikes in stress regimes
- Senior tranche losses that were priced as very remote become much more plausible
- Jump-to-default on concentrated names

### 50.10.3 Delta-Neutral Correlation Trades

To isolate correlation exposure from spread exposure, traders construct **delta-neutral** positions:

**Step 1:** Calculate tranche deltas (spread sensitivity to index)
- Define the spread bump object and the rebuild rule (what stays fixed vs recomputed)
- Compute deltas for both legs under the same methodology

**Step 2:** Size positions to offset index spread exposure
- For every USD 1 of equity sold, buy $k$ of senior (where $k$ = equity delta / senior delta)

**Step 3:** Monitor and rebalance
- Deltas change as spreads move
- Rebalancing required to maintain neutrality

### 50.10.4 The Carry-Correlation Tradeoff

Correlation packages are rarely “pure correlation”. Even if you spread‑delta‑hedge, P&L is usually a mixture of:
- premium income vs protection payments (carry / realized loss),
- dependence/skew shifts under the desk’s model, and
- basis/model risk from mapping and interpolation.

Junior tranches respond most to changes in the probability of **very low** portfolio loss outcomes, while senior tranches are driven by **tail** scenarios. This is why equity‑vs‑senior packages can have strong dependence exposure even when spread delta is approximately hedged.

### 50.10.5 How Traders Quote Correlation

> **Desk Reality:** In some tranche markets, participants quote a **model-implied correlation** (compound or base) rather than quoting only a spread/upfront.
> **Common break:** Two parties can “agree on 22% base correlation” but still disagree on PV/risk if they use different calibration sets, recovery assumptions, interpolation rules, or bump/rebuild methodologies.
> **What to check:** Always state the quoting convention (compound vs. base), the strike/maturity, and the calibration + interpolation method; verify by repricing that the tranche PV matches the quote under the agreed setup.

---

## 50.11 Model Risk: What Breaks and How to Stress

### 50.11.1 Why Quoting Models Can Mislead About Tail Risk

Base-correlation-style frameworks are useful because they provide a shared quoting language. The same features that make them convenient also create model risk:

1. **Dependence is under-identified:** tranche prices pin down a *family* of joint-default structures; “implied correlation” is a parametrization choice, not a uniquely determined economic object.
2. **Tail assumptions dominate senior risk:** senior/super-senior tranches are sensitive to joint-extreme scenarios; models with weak tail dependence can understate that risk.
3. **Methodology dependence:** the implied parameter depends on recovery assumptions, marginal curves, discounting choices, calibration set, and interpolation/extrapolation rules.
4. **Interpolation non-locality:** depending on the scheme, bumping one quoted node can move interpolated values elsewhere, changing hedges in unintuitive ways.

> **Desk Reality:** Treat “correlation” as a **state variable attached to a methodology**, not as a portable statistic.
> **Common break:** Teams compare “base correlation” across systems without aligning inputs (curves/recovery), calibration set, and interpolation rules.
> **What to check:** Agree on a repricing stack end-to-end; then stress dependence (level and shape) and verify that tranchelet spreads and implied densities remain economically consistent.

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
- Simulate large dependence shifts and steepening/flattening of the correlation skew

**Recovery stress:**
- Test sensitivity to recovery assumptions ($R = 20\\%$ vs. $40\\%$ vs. $60\\%$)

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

## 50.12 Bespoke Tranches and Mapping (High-Level)

For portfolios that are not standard indices, practitioners use **mapping techniques** to borrow correlation from liquid index tranches.

The purpose of mapping is to choose a “standard” strike $K_s^{\star}$ on a liquid index so that its base correlation $\rho_s(K_s^{\star})$ can be used as an input when pricing a bespoke tranche.

**Key mapping approaches:**

1. **No-mapping (direct):** Use index base correlation at same strike
   - Simple but ignores portfolio differences

2. **ATM-style mapping:** Match ratio of strike to expected loss
   $$\frac{K_B}{\mathbb{E}[L_B]} = \frac{K_s}{\mathbb{E}[L_s]}$$

3. **TLP (tranche loss proportion) mapping:** Match ratio of tranche loss to portfolio loss
   - Useful when “moneyness” differs across portfolios

> **Desk Reality:** Bespoke “correlation” is usually hedged indirectly using liquid standard tranches.
> **Common break:** Mapping assumptions can break exactly in the scenarios that matter (stress regimes), creating basis risk.
> **What to check:** Report bespoke risk as “equivalent notionals” in liquid tranches *and* validate with scenario shocks (spread level, skew shape, recovery, tail dependence).

### 50.12.1 Bespoke Risk Management

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
- Portfolio: 125 names, 5y maturity, $R = 40\\%$
- Discount rate: 5% continuous

| Tranche | Market Spread (bps) | Upfront |
|---------|---------------------|---------|
| 0–3% | 500 running | 35% |
| 3–7% | 250 running | — |
| 7–10% | 85 running | — |

**Step 1: Calibrate $\rho(3\\%)$**

Using one-factor Gaussian copula, find $\rho$ such that equity tranche PV = 0 at 35% upfront + 500bp running.

Iterating:
- $\rho = 10\\%$: Model upfront = 42% (too high)
- $\rho = 20\\%$: Model upfront = 33% (too low)
- $\rho = 18\\%$: Model upfront = 35.1% ≈ market

**Result:** $\rho(3\\%) = 18\\%$

**Step 2: Calibrate $\rho(7\\%)$**

Holding $\rho(3\\%) = 18\\%$ fixed, find $\rho(7\\%)$ such that [3%, 7%] tranche prices at 250bp.

Compute [3%, 7%] ETL as:
$$ETL_{3,7} = \frac{\psi(7\\%, \rho(7\\%)) - \psi(3\\%, \rho(3\\%))}{0.04}$$

Iterating:
- $\rho(7\\%) = 25\\%$: Implied spread = 230bp (too low)
- $\rho(7\\%) = 30\\%$: Implied spread = 255bp (close)
- $\rho(7\\%) = 29\\%$: Implied spread = 249bp ≈ market

**Result:** $\rho(7\\%) = 29\\%$

**Step 3: Calibrate $\rho(10\\%)$**

Similarly: **Result:** $\rho(10\\%) = 38\\%$

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
$$\rho(4\\%) = 18\\% + \frac{1}{4}(29\\% - 18\\%) = 20.75\\%$$
$$\rho(5\\%) = 18\\% + \frac{2}{4}(29\\% - 18\\%) = 23.5\\%$$
$$\rho(6\\%) = 18\\% + \frac{3}{4}(29\\% - 18\\%) = 26.25\\%$$

**Compute tranchelet ETLs:**
- $\psi(4\\%, 20.75\\%) = 2.85\\%$
- $\psi(5\\%, 23.5\\%) = 3.80\\%$
- $\psi(6\\%, 26.25\\%) = 4.70\\%$

**Tranchelet spreads (using simplified spread ≈ ETL / risky PV01):**
- [4%, 5%]: ETL = $(3.80 - 2.85)/0.01 = 95\\%$ → spread ≈ 1900bp
- [5%, 6%]: ETL = $(4.70 - 3.80)/0.01 = 90\\%$ → spread ≈ 1800bp

**Monotonicity check:** [5%, 6%] has more subordination but lower spread than [4%, 5%]. ✓ Passes.

**Sensitivity:** If interpolation were steeper, [5%, 6%] could have *higher* spread—an arbitrage signal.

---

### Example 6: Recovery Sensitivity

Using Example 4 setup with $\rho(3\\%) = 18\\%$ calibrated at $R = 40\\%$.

**Question:** What happens to implied $\rho(3\\%)$ if we assume $R = 50\\%$?

**At $R = 50\\%$:** Higher recovery means lower loss-given-default, so the same tranche spread requires *lower* correlation to generate enough expected loss.

Recalibrating:
- With $R = 50\\%$, $\rho(3\\%) = 14\\%$ matches the 35% upfront

**Key insight:** Recovery and correlation are not independently identifiable from a single tranche quote. Higher assumed recovery implies lower implied correlation.

---

### Example 7: Correlation 01 Calculation

**Setup:** Use calibrated base correlations from Example 4.

**Equity [0–3%] correlation 01:**
1. Compute PV at $\rho(3\\%) = 18\\%$: PV = 0 (calibrated)
2. Bump to $\rho(3\\%) = 19\\%$: recompute ETL and PV
3. $\text{Corr01} = \text{PV}(19\\%) - \text{PV}(18\\%)$

Result (sign): Corr01 $>0$ (short protection gains when correlation rises)

**Senior [7–10%] correlation 01:**
1. Bump both $\rho(7\\%)$ and $\rho(10\\%)$ by 1%
2. Recompute tranche PV

Result (sign): Corr01 $\lt 0$ (short protection loses when correlation rises)

**Sign confirmation:** Opposite signs across capital structure, as expected.

---

### Example 8: Tail Dependence Comparison

**Question:** Compare the probability of joint extreme events under Gaussian vs. t-copula.

**Setup:** Two names with pairwise correlation $\rho = 0.5$.

**Gaussian copula ($\nu = \infty$):**
- Tail dependence coefficients: $\lambda_{\mathrm{U}}=\lambda_{\mathrm{L}}=0$
- Interpretation: as you move deeper into the tail, $\Pr(\text{name 2 extreme} \mid \text{name 1 extreme})$ tends to 0 (asymptotic tail independence)

**t-copula with $\nu = 4$:**
Tail dependence coefficient:
$$\lambda_{\mathrm{U}}=\lambda_{\mathrm{L}}=2 t_5\left(-\sqrt{\frac{5(1-\rho)}{1+\rho}}\right)=2t_5(-1.29)\approx 0.253$$

Interpretation: in the extreme tail, conditioning on one name being extreme leaves a non-trivial chance (about 25%) that the other name is also extreme.

**Implication for tranches:** A super-senior tranche that appears safe under Gaussian assumptions can have substantially higher risk under tail-dependent models. This is one reason senior tranches are highly sensitive to dependence assumptions.

---

### Example 9: Delta-Neutral Correlation Trade

**Setup:** CDX 5Y, correlation at mid-levels

**Position:**
- Sell USD 10mm notional equity protection (0-3%)
- Buy senior protection (10-15%)

**Illustrative deltas (toy):**
- Equity delta: ~20x (highly leveraged)
- Senior delta: ~2x

**Delta-neutral sizing:**
- Equity DV01: $10mm × 20 = $200mm equivalent
- Senior notional needed: $200mm / 2 = $100mm

**Resulting position:**
- Sell USD 10mm 0-3% protection
- Buy USD 100mm 10-15% protection
- Spread-neutral but long correlation

**P&L for +5% correlation move:**
- Equity: +USD 500k (gains from lower ETL)
- Senior: +USD 250k (gains from higher ETL, but you're long protection)
- Net: +USD 750k

---

### Example 10: High-Dependence Scenario (Toy)

**Setup:** Portfolio with 5% expected loss, $R = 40\\%$

**Normal environment ($\rho = 25\\%$):**

| Tranche | ETL |
|---------|-----|
| 0-3% | 98% |
| 3-7% | 35% |
| 10-15% | 2% |
| 15-30% | 0.1% |

**High-dependence environment ($\rho = 60\\%$):**

| Tranche | ETL |
|---------|-----|
| 0-3% | 75% |
| 3-7% | 28% |
| 10-15% | 12% |
| 15-30% | 5% |

**Key observations:**
- Equity tranche ETL *decreased* (more "no loss" scenarios)
- Super-senior ETL *increased 50×* (tail risk materialized)
- This is a standard stress pattern: higher dependence can *reduce* equity ETL while *increasing* senior ETL.

---

## 50.14 Practical Notes

### 50.14.1 Input Checklist

| Category | Required Inputs |
|----------|-----------------|
| **Portfolio** | Constituents, notional weights, maturity |
| **Marginals** | CDS curves for each name (survival probabilities) |
| **Recovery** | Fixed or stochastic; must be stated (e.g., 40% as an illustrative assumption) |
| **Tranches** | Attachment/detachment, coupon, day count |
| **Discount** | Discount curve assumption must be stated (e.g., OIS) |
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

## Summary
1. Tranche PV depends on the **distribution** of portfolio loss because the tranche loss function is nonlinear in $L(T)$.
2. Holding marginals fixed, increasing dependence reshapes the loss distribution (more “all-or-nothing” mass) without changing $\mathbb{E}[L(T)]$.
3. Equity and senior tranches can have **opposite dependence sensitivities**; the **sign** depends on whether you are long or short protection.
4. Copulas (via Sklar’s theorem) separate marginal default behavior from the dependence assumption used to generate joint outcomes.
5. The one-factor Gaussian latent-variable model gives conditional independence and a simple conditional default probability; tranche ETLs follow by integrating over the factor.
6. Gaussian copulas are asymptotically tail independent; t-copulas have positive tail dependence, which matters most for senior tranches.
7. The LHP/Vasicek limit provides fast analytical approximations that are useful for intuition and scenario analysis.
8. Compound correlation (one flat $\rho$ per tranche) can have multiple roots for mezzanine tranches and is not a consistent “smile” model across strikes.
9. Base correlation (one $\rho(K)$ per detachment) is a quoting language; tranche ETL is computed from two base-tranche ETLs $\psi(T,K)$.
10. Interpolation/extrapolation rules can imply negative densities or non-monotone tranchelet spreads; diagnostics belong in every implementation.
11. Corr01 is methodology-dependent: always state bump object, bump size, units, and rebuild rule, and validate with scenario shocks (not only local bumps).
12. Bespoke mapping converts liquid index “correlation” into bespoke inputs, creating basis/model risk that must be stress-tested.

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Tranche loss mapping | $L(T;A,D)=\frac{\min(L(T),D)-\min(L(T),A)}{D-A}$ | Nonlinearity drives dependence sensitivity |
| Copula | A joint CDF written in terms of marginal CDFs + a dependence function | Separates marginals from dependence |
| Sklar’s theorem | Joint CDF = copula of marginals; unique if marginals continuous | Foundation for copula construction |
| One-factor Gaussian latent variable | $Z_i=\sqrt{\rho}\\,X+\sqrt{1-\rho}\\,\varepsilon_i$ | Workhorse dependence parametrization |
| Conditional default probability | $p_i(T\mid X=x)=\Phi\\!\left(\frac{a_i(T)-\sqrt{\rho}\\,x}{\sqrt{1-\rho}}\right)$ | Turns $N$-D dependence into 1-D integration |
| Tail dependence | Limit probability of joint extremes; summarized by $\lambda_{\mathrm{U}},\lambda_{\mathrm{L}}$ | Senior risk is tail-sensitive |
| LHP/Vasicek limit | Large homogeneous portfolio approximation | Fast intuition + approximations |
| Base-tranche ETL | $\psi(T,K)=\mathbf{E}_{\rho(K)}[\min(L(T),K)]$ | Core object for base correlation + density checks |
| Base correlation | Assigns $\rho(K)$ to each base tranche $[0,K]$ | Market quoting language |
| Tranche ETL under base correlation | $\mathbb{E}[L(T;A,D)]=\frac{\psi(T,D)-\psi(T,A)}{D-A}$ | “Difference of base tranches” decomposition |
| Implied loss density | $f(K)=-\frac{\partial^2\psi(T,K)}{\partial K^2}$ | Negative $f$ indicates arbitrage-like artifacts |
| Corr01 | $PV(\rho+0.01)-PV(\rho)$ for the stated position | Dependence risk in currency per 1% |

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $A,D$ | attachment / detachment | fractions of portfolio notional (e.g., 3% = 0.03) |
| $w=D-A$ | tranche width | fraction of portfolio notional |
| $L(T)$ | portfolio loss at $T$ | fraction of portfolio notional |
| $L(T;A,D)$ | tranche loss fraction | fraction of tranche notional, in $[0,1]$ |
| $Q(t;A,D)$ | tranche survival curve | $Q(t;A,D)=\mathbb{E}[1-L(t;A,D)]$ |
| $\psi(T,K)$ | base-tranche ETL | $\psi(T,K)=\mathbf{E}_{\rho(K)}[\min(L(T),K)]$ |
| $f(K)$ | implied loss density | $f(K)=-\partial^2\psi(T,K)/\partial K^2$ |
| $\rho$ | dependence parameter | model parameter; specify compound vs base node |
| $\Phi,\Phi^{-1}$ | standard normal CDF / inverse | unitless |
| $t_\nu,t_\nu^{-1}$ | t CDF / inverse | unitless |
| $\lambda_{\mathrm{U}},\lambda_{\mathrm{L}}$ | tail dependence coefficients | unitless |
| $PV(\rho)$ | PV of the stated position | currency |
| Corr01 | dependence sensitivity | $PV(\rho+0.01)-PV(\rho)$, currency per 1% |

## Flashcards

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
| 9 | Does Gaussian copula have tail dependence? | No ($\lambda_{\mathrm{U}} = \lambda_{\mathrm{L}} = 0$) |
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
| 20 | What is Corr01? | $PV(\rho+0.01)-PV(\rho)$ for the stated position; currency per 1% dependence bump |
| 21 | Why do equity and senior have opposite correlation 01 signs? | Different exposure to loss distribution extremes |
| 22 | What is the LHP model? | Large Homogeneous Portfolio limit giving closed-form loss distribution |
| 23 | Why is Gaussian copula still used despite known tail dependence failures? | It's the market quoting convention; everyone speaks the same language; alternatives don't necessarily hedge better |
| 24 | What is a "long correlation" position? | Sell equity protection + buy senior protection (delta-neutral); profits if correlation increases |
| 25 | What does tail dependence measure and what is it for Gaussian copula? | Probability of joint extreme events; Gaussian copula has zero tail dependence |

## Mini Problem Set

1. (Compute) Prove the bounds $L(T;A,D)\in[0,1]$.
2. (Compute) For $A=3\\%$, $D=7\\%$, compute tranche loss fractions when $L(T)=2\\%$, $5\\%$, $10\\%$.
3. (Compute) Ten names have PD $=5\\%$. Under independence, compute $\Pr(K\ge 3)$, where $K$ is the number of defaults.
4. (Compute) For the same PD, under perfect dependence (all-default-or-none), compute $\Pr(K\ge 3)$.
5. (Compute) Compute the default threshold $a=\Phi^{-1}(0.01)$.
6. (Concept) Starting from $Z_i=\sqrt{\rho}X+\sqrt{1-\rho}\varepsilon_i$, derive $p_i(T\mid X=x)$.
7. (Concept) Explain why $\rho=0$ gives independence in the one-factor model.
8. (Concept) Explain why $\rho\to 1$ gives “all-or-nothing” clustering.
9. (Compute) Using $\lambda = 2t_{\nu+1}\\!\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)$, compute $\lambda$ for a t-copula with $\rho=0.7$, $\nu=4$.
10. (Desk) Name two diagnostics for base-correlation interpolation artifacts and what “failure” looks like.
11. (Concept) For **short protection**, explain why equity and super-senior can have opposite Corr01 signs.
12. (Compute) A tranche has $PV(\rho)=USD 1.20$mm and $PV(\rho+1\%)=USD 1.26$mm (same methodology). Compute Corr01.

### Solution Sketches (Selected)
2. $L=2\\%\lt A\Rightarrow L(T;A,D)=0$. $L=5\\%\Rightarrow (0.05-0.03)/0.04=0.5$. $L=10\\%\gt D\Rightarrow L(T;A,D)=1$.
3. $\Pr(K\ge 3)=1-\sum_{k=0}^2 {10\choose k}0.05^k0.95^{10-k}\approx 0.0115$ (about 1.15%).
4. Perfect dependence implies $K\in\\{0,10\\}$ with $\Pr(K=10)=0.05$. Hence $\Pr(K\ge 3)=0.05$.
5. $a=\Phi^{-1}(0.01)\approx -2.3263$.
9. With $\nu=4$, $\nu+1=5$: $\lambda=2t_{5}\\!\left(-\sqrt{5\times 0.3/1.7}\right)\approx 0.3907$.
12. $\mathrm{Corr01}=PV(\rho+0.01)-PV(\rho)=USD 0.06\text{mm}=USD 60{,}000$ per 1% dependence bump.

## References
- (Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, “Gaussian copula and default clustering”; “LHP model”; “Compound correlation”; “Base correlation”; “Implied loss density”; “ETL interpolation”.)
- (Benhamou et al., *Implementing Models in Quantitative Finance*, “Copula Functions / Sklar (1959)”; “Upper and lower tail dependence”.)
- (McNeil, Frey, Embrechts, *Quantitative Risk Management*, Examples on Gaussian vs t-copula tail dependence.)
- (N. Privault, *Financial Risk Analytics*, Chapter on synthetic CDOs / tranche premium and protection legs.)
- (John C. Hull, *Options, Futures, and Other Derivatives*, sections on synthetic CDOs and implied correlation measures.)
- (David X. Li, “On Default Correlation: A Copula Function Approach”, 2000.)
- (*Data Analytics for Credit and Related Risk*, section on correlated default times and Gaussian copula construction.)
