# Chapter 49: Tranche Core Concepts — Expected Tranche Loss and Present Value

---

## Introduction

How much of the 3–7% tranche do we expect to lose over five years? This deceptively simple question lies at the heart of tranche pricing. The answer—the Expected Tranche Loss (ETL)—is the central object that drives both the protection leg and the premium leg of any CDO tranche. Get the ETL wrong, and tranche pricing collapses entirely: you misjudge the fair spread, misprice the protection value, and create arbitrage opportunities across the capital structure.

Chapter 48 introduced the tranche product structure: attachment points, detachment points, and the nonlinear mapping from portfolio losses to tranche losses. This chapter takes the next step by showing how to compute the expected value of that tranche loss—the ETL—and use it to price the two legs of a tranche trade.

## Learning Objectives
- Define portfolio loss, tranche loss, and expected tranche loss (ETL), with explicit units.
- Translate a tranche quote (running spread and/or fixed coupon + upfront) into expected cashflows.
- Price the protection leg and premium leg from an ETL term structure (or tranche survival curve $Q$).
- Compute and interpret tranche `RPV01`, par spread, and upfront (with clear sign conventions).
- Apply basic no-arbitrage checks to ETL surfaces (bounds, monotonicity, concavity, conservation).

Prerequisites: [Chapter 48 — CDO / Tranche Products — What They Are (Product Map)](chapters/chapter_48_cdo_tranche_products_product_map.md); [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md); [Chapter 42 — Bootstrapping a CDS Survival Curve from Market Quotes](chapters/chapter_42_bootstrapping_cds_survival_curve.md) (optional: [Chapter 41 — CDS Indices — Mechanics, Coupons, Rolls](chapters/chapter_41_cds_indices_mechanics_coupons_rolls.md)).

Follow-on: [Chapter 50 — Correlation and Tranche Pricing Frameworks](chapters/chapter_50_correlation_tranche_pricing_frameworks.md); [Chapter 51 — Tranche Risk — Tranche PV01, Correlation Risk, and Jump-to-Default Clustering](chapters/chapter_51_tranche_risk.md); [Chapter 52 — Credit Trading Strategies (Risk + Instrument + Hedge)](chapters/chapter_52_credit_trading_strategies.md).

A key desk-level insight is that tranche pricing can be written with essentially the same premium-leg / protection-leg algebra as CDS pricing once you work with an appropriate “survival” object for the tranche:

- Replace the issuer survival probability with a **tranche survival curve** $Q(t;A,D)$ (the expected surviving fraction of tranche notional).
- In the CDS-style algebra, treat tranche losses as **principal write-down** (a “zero recovery on tranche notional” analogy), because losses show up as reductions in outstanding tranche notional.

Once you have $Q(t;A,D)$ (or equivalently the ETL term structure), the CDS PV machinery carries over almost verbatim.

For middle-office professionals transitioning to trading desks, understanding ETL computation bridges two worlds. Your valuation reports already show tranche fair values, but now you'll understand how those numbers are derived. When spreads move and P&L changes, you'll trace the chain from loss distribution shifts to ETL changes to tranche value adjustments. This knowledge transforms you from a report consumer into someone who can diagnose pricing discrepancies and anticipate how model changes flow through to the trading book.

The chapter proceeds as follows. Section 49.1 establishes the core definitions: portfolio loss as the state variable, the tranche loss function, and the critical ETL quantity. Section 49.2 derives the ETL from both discrete and continuous loss distributions, showing the computational mechanics including the recursion algorithm, the LHP closed-form model, and Monte Carlo approaches. Section 49.3 develops the protection leg PV as discounted expected loss increments. Section 49.4 derives the premium leg PV using the average-notional approximation. Section 49.5 combines these to define the tranche RPV01, par spread, mark-to-market value, and upfront payment conventions. Sections 49.6–49.7 cover the term structure of ETL and no-arbitrage constraints. Section 49.8 provides implementation guidance including accuracy comparisons and validation checklists.

---

## 49.1 Core Definitions: From Portfolio Loss to Expected Tranche Loss

### 49.1.1 Portfolio Loss as the State Variable

The foundation of tranche pricing is a single state variable: the portfolio cumulative loss fraction $L(t)$.

A general (weighted) definition is:

$$L(t) = \sum_{i=1}^{N_c} w_i (1-R_i)\mathbf{1}_{\tau_i \le t}, \qquad \sum_{i=1}^{N_c} w_i = 1$$

If exposures are equal ($w_i = 1/N_c$), this reduces to the familiar equal‑weight formula:

$$L(t) = \frac{1}{N_c} \sum_{i=1}^{N_c} (1-R_i) \mathbf{1}_{\tau_i \le t}$$

The portfolio loss is bounded: $0 \le L(t) \le L_{\max}$, where $L_{\max} = \sum_i w_i(1-R_i)$ (and equals $1-\bar R$ in the common‑recovery simplification).

**Why this matters for tranche pricing:** The tranche loss function $\text{TL}(L)$ is a deterministic function of $L(t)$. Any model that produces the distribution of $L(t)$—whether a simple binomial, a Gaussian copula, or a sophisticated dynamic model—immediately implies the distribution of tranche losses. The modeling challenge reduces to characterizing one state variable rather than tracking defaults credit-by-credit.

> **Desk Reality: What a Single Default Means**
>
> Consider a standard index tranche on 125 names with 40% recovery. A single default produces:
>
> $$\Delta L = \frac{1-0.40}{125} = 0.48\%$$
>
> For the **0-3% equity tranche** (width 3%), this single default consumes 16% of the tranche width! For the **15-30% senior tranche**, this same default produces *zero* tranche loss—the senior tranche doesn't start losing until portfolio losses exceed 15%, requiring roughly 32 defaults.
>
> This asymmetric exposure is why equity and senior tranches can trade with radically different risk profiles (and therefore very different pricing) despite referencing the same portfolio.

### 49.1.2 Tranche Loss Function: The Nonlinear Mapping

A tranche with attachment point $A$ (also written $K_1$) and detachment point $D$ (also written $K_2$) absorbs portfolio losses only within its layer. O'Kane defines the fractional tranche loss at horizon $T$ as:

$$L(T, K_1, K_2) = \frac{\max(L(T) - K_1, 0) - \max(L(T) - K_2, 0)}{K_2 - K_1}$$

This normalized loss lies in $[0,1]$: zero when portfolio losses stay below attachment, unity when losses exceed detachment. In portfolio-notional units (unnormalized), the tranche loss amount is:

$$\boxed{\text{TL}(L) = \min(\max(L - A, 0), W)}$$

where $W = D - A$ is the tranche width. An equivalent and often useful form is:

$$\text{TL}(L) = \min(L, D) - \min(L, A)$$

This identity reveals that the tranche loss equals the difference between two "equity" payoffs—a structure we exploit when discussing base correlation in Chapter 50.

> **Analogy: The Bull Call Spread**
>
> In option terms, the Tranche Loss is a **Bull Call Spread** on the Portfolio Loss.
> - **Long Call @ A**: You start losing when losses exceed A.
> - **Short Call @ D**: You stop losing (reach max loss) when losses exceed D.
>
> Note the perspective flips for the **Protection Seller**:
> - **Short Call @ A**: Seller pays losses above A.
> - **Long Call @ D**: Seller stops paying above D.
> - This is a **Short Call Spread** (or a **Credit Put Spread** in option lingo). You collect premium and hope the underlying (Losses) stays low.

**The tranche outstanding notional**, which determines premium payments, is simply:

$$\boxed{\text{ON}(L) = W - \text{TL}(L)}$$

As portfolio losses accumulate, the tranche outstanding shrinks, reducing subsequent premium payments.

### 49.1.3 The "Hockey Stick" Effect: Tranche Loss by Number of Defaults

The leverage inherent in tranching becomes vivid when we tabulate tranche losses for different default scenarios. Consider a 125-name portfolio with 40% recovery (so each default produces 0.48% portfolio loss):

| # Defaults | Portfolio Loss | 0-3% TL | 3-7% TL | 7-10% TL | 10-15% TL | 15-30% TL |
|:----------:|:--------------:|:-------:|:-------:|:--------:|:---------:|:---------:|
| 0 | 0.00% | 0.00% | 0.00% | 0.00% | 0.00% | 0.00% |
| 1 | 0.48% | 0.48% | 0.00% | 0.00% | 0.00% | 0.00% |
| 3 | 1.44% | 1.44% | 0.00% | 0.00% | 0.00% | 0.00% |
| 5 | 2.40% | 2.40% | 0.00% | 0.00% | 0.00% | 0.00% |
| 7 | 3.36% | **3.00%** | 0.36% | 0.00% | 0.00% | 0.00% |
| 10 | 4.80% | **3.00%** | 1.80% | 0.00% | 0.00% | 0.00% |
| 15 | 7.20% | **3.00%** | **4.00%** | 0.20% | 0.00% | 0.00% |
| 20 | 9.60% | **3.00%** | **4.00%** | 2.60% | 0.00% | 0.00% |
| 32 | 15.36% | **3.00%** | **4.00%** | **3.00%** | 5.00% | 0.36% |

The **cliff effect** is stark: under this simplified arithmetic (equal weights, 40% recovery), the 0–3% equity tranche is exhausted after ~7 defaults (bold entries show capped losses), while the 15–30% super‑senior doesn’t begin losing until ~32 defaults.

### 49.1.4 Expected Tranche Loss: The Central Object

The Expected Tranche Loss (ETL) is the expectation of the tranche loss function under the risk-neutral measure:

$$\boxed{\mathrm{ETL}(t) = E[\text{TL}(L(t))] = E[\min(L(t), D) - \min(L(t), A)]}$$

**Check (bounds + units):** $\mathrm{ETL}(t)$ here is a **portfolio-notional fraction**. It is bounded by tranche width: $0 \le \mathrm{ETL}(t) \le W:=D-A$. The expected outstanding notional $\text{EON}(t)=W-\mathrm{ETL}(t)$ is also a portfolio-notional fraction, bounded by $0\le \text{EON}(t)\le W$. If you want “percent of tranche notional,” divide by $W$.

For equity (base) tranches with $A = 0$, O'Kane introduces the notation $\psi(T, K)$ for the ETL curve:

$$\psi(T, K) = E[\min(L(T), K)]$$

This is "the expected loss at time $T$ of an equity tranche with width $K$ seen from time 0." Any mezzanine or senior tranche's ETL can be written as the difference of two equity ETLs:

$$\mathrm{ETL}(T; A, D) = \psi(T, D) - \psi(T, A)$$

This decomposition is the foundation of the base correlation framework discussed in Chapter 50.

**The expected outstanding notional** is the complement:

$$\boxed{\text{EON}(t) = W - \mathrm{ETL}(t)}$$

**Check (toy):** in Worked Example 49.1 for the $[3\%,7\%]$ tranche, $W=4\%$ and $\mathrm{ETL}=1.0\%$, so $\text{EON}=3.0\%$. Interpreting as “tranche fraction,” the expected surviving tranche notional is $3/4 = 75\%$ of face at that horizon.

### 49.1.5 Tranche Survival and the CDS Analogy

The **tranche survival probability** $Q(t; A, D)$ measures the expected surviving fraction of tranche notional:

$$Q(t; A, D) = E[1 - L_{\text{tr}}(t)] = \frac{\text{EON}(t)}{W}$$

where $L_{\text{tr}}(t) = \text{TL}(L(t))/W$ is the normalized tranche loss. A key observation (made explicitly in O’Kane’s derivations) is that tranche pricing admits a clean CDS analogy once you express everything in terms of the tranche survival curve $Q(t;A,D)$.

The CDS analogy is:

- Use $Q(t;A,D)$ in place of the issuer survival probability.
- Set “recovery” to zero in the CDS pricing algebra because tranche losses are literally reductions in outstanding notional.

So **once we have the tranche survival curve** (or equivalently the ETL term structure), we can reuse CDS‑style premium/protection leg PV formulas with minimal changes. That is why ETL computation is the critical modeling task.

---

## 49.2 Computing ETL from Loss Distributions

### 49.2.1 ETL from a Discrete Loss Distribution

In practice, portfolio loss distributions are often discrete—either because the portfolio has a finite number of credits, or because we discretize a continuous distribution for computation. Given scenarios $\\{L_j\\}$ with probabilities $\\{p_j\\}$:

$$\mathrm{ETL}(T) = \sum_j p_j \cdot \text{TL}(L_j) = \sum_j p_j \cdot [\min(L_j, D) - \min(L_j, A)]$$

**Worked Example 49.1 (Discrete ETL Computation):**

Consider a tranche $[3\%, 7\%]$ with width $W = 4\%$ and the following horizon loss distribution:

| Portfolio Loss $L$ | Probability | $\text{TL}(L)$ | $p \times \text{TL}$ |
|:------------------:|:-----------:|:--------------:|:--------------------:|
| 0% | 0.50 | 0 | 0 |
| 2% | 0.20 | 0 | 0 |
| 6% | 0.20 | 3% | 0.6% |
| 15% | 0.10 | 4% (capped) | 0.4% |

$$\mathrm{ETL}(T) = 0 + 0 + 0.6\% + 0.4\% = 1.0\%$$

The expected outstanding is $\text{EON}(T) = 4\% - 1\% = 3\%$, and the tranche survival is $Q(T) = 3\%/4\% = 75\%$.

### 49.2.2 ETL from a Continuous Loss Distribution

When the portfolio loss has a continuous density $f(L)$, the ETL becomes an integral:

$$\mathrm{ETL}(T) = \int_0^{L_{\max}} \text{TL}(\ell) \cdot f(\ell) \\, d\ell$$

Using the piecewise structure of $\text{TL}(L)$:

$$\mathrm{ETL}(T) = \int_A^D (\ell - A) f(\ell) \\, d\ell + \int_D^{L_{\max}} W \cdot f(\ell) \\, d\ell$$

The first integral captures partial losses within the tranche layer; the second captures the full wipeout probability times $W$.

**Worked Example 49.2 (ETL from Continuous Density):**

Suppose portfolio losses follow a beta distribution on $[0, 60\%]$ with parameters such that the density is $f(L) = 20L(0.6-L)/0.6^3$ for $L \in [0, 0.6]$ (a symmetric "hill" shape with mean 30%).

For tranche $[3\%, 7\%]$:

$$\mathrm{ETL} = \int_{0.03}^{0.07} (\ell - 0.03) f(\ell) \\, d\ell + \int_{0.07}^{0.60} 0.04 \cdot f(\ell) \\, d\ell$$

The first integral captures partial losses; the second captures full wipeout probability times $W$. While this specific integral requires numerical evaluation, the structure is general: **ETL combines the "interior" expected loss with the probability mass above the detachment point.**

### 49.2.3 Conditional Independence and the Factor Model

O'Kane devotes Chapter 18 to computing the loss distribution efficiently. The key insight is **conditional independence**: in a one-factor Gaussian copula model, credits are independent conditional on the common factor $Z$. The conditional loss distribution $f(L|Z)$ is binomial (for homogeneous portfolios) or multinomial (for heterogeneous ones), and the unconditional distribution is obtained by integration:

$$f(L) = \int_{-\infty}^{\infty} f(L|Z) \phi(Z) \\, dZ$$

where $\phi(Z)$ is the standard normal density.

> **Beginner Bridge: What "Conditional Independence" Means**
>
> Imagine two companies, Ford and GM. Their defaults are correlated because both depend on the economy. But *given* a specific economic state (say, "mild recession"), their remaining default risks are independent—Ford's factory fire doesn't cause GM to default.
>
> The factor $Z$ captures this common driver. When $Z$ is very negative (severe recession), both Ford and GM have high conditional default probabilities. When $Z$ is positive (boom), both have low probabilities. But conditional on $Z$, their defaults are independent coin flips.
>
> This structure—correlated through a factor but conditionally independent—makes computation tractable.

### 49.2.4 The Recursion Algorithm in Detail

O'Kane presents the recursion algorithm in Chapter 18.3.1 as the foundation for exact loss distribution computation. The algorithm builds the conditional loss distribution iteratively by adding one credit at a time.

**Setup:** Define the loss distribution after processing $k$ credits as $P_k(j)$, the probability that cumulative loss equals $j$ loss units, where a "loss unit" is typically the smallest loss amount in the portfolio (e.g., if all credits have equal notional and recovery, one loss unit = $(1-R)/N$).

**Recursion formula:** When adding credit $k+1$ with conditional default probability $p_{k+1}(T|Z)$ and loss amount $\ell_{k+1}$ loss units:

$$\boxed{P_{k+1}(j) = (1 - p_{k+1}) P_k(j) + p_{k+1} P_k(j - \ell_{k+1})}$$

**Algorithm:**
```
Initialize: P_0(0) = 1, P_0(j) = 0 for j > 0
For k = 1 to N_c:
    For j = max_loss down to ℓ_k:
        P_k(j) = (1 - p_k) P_{k-1}(j) + p_k P_{k-1}(j - ℓ_k)
    For j = 0 to ℓ_k - 1:
        P_k(j) = (1 - p_k) P_{k-1}(j)
```

**Complexity:** For a portfolio of $N_c$ credits where the maximum number of loss units is $N_U$, the algorithm requires $O(N_c \times N_U)$ operations per factor value. For homogeneous portfolios with unit losses, $N_U = N_c$, giving $O(N_c^2)$ complexity.

**Why process credits in reverse order for $j$?** O'Kane notes that processing from high to low loss values allows the algorithm to update the distribution in-place, using only one array of size $N_U$ rather than storing both $P_{k-1}$ and $P_k$.

### 49.2.5 Loss Unit Granularity and Recovery Effects

The choice of loss unit can significantly impact computational cost when you discretize losses. O’Kane shows that the “loss unit” grid can **blow up discontinuously** when recoveries (or notionals) are not aligned.

One illustrative experiment: take 125 credits, set 124 recoveries to 40%, and vary the recovery of the remaining name. A seemingly small change (e.g., one name moving from 40% to 39% recovery) can increase the required number of loss units from 125 to 7501; even moving to 35% can imply around 1500 loss units. The inner loop then becomes much more expensive because it runs over loss units.

**Why this happens:** In the recursion method, you want a discretization unit $u$ such that each name’s loss amount $l_i$ is an integer multiple of $u$ (so you can represent the loss distribution on an integer grid). If all names share identical notionals and recoveries, this is easy and the grid size is modest. But if one name has a slightly different recovery/weight, the effective “GCD” of the loss amounts can become very small, and the number of grid points (loss units) can jump dramatically.

**The Andersen et al. algorithm:** O'Kane describes an adaptive approach that caps the discretization error at a tolerance $\epsilon$ while minimizing the number of loss units. The algorithm searches for the smallest loss unit $u$ such that:

$$\left|l_j - \text{int}(l_j / u) \cdot u\right| \lt \epsilon$$

for all credits, where $l_j = F_j(1-R_j)$ is the loss amount for credit $j$.

> **Desk Reality: "Round Number" Recoveries**
>
> Practitioners often use stylized recovery assumptions (e.g., 40% on IG indices) in tranche models. One pragmatic reason is computational convenience in discretized loss‑distribution methods: “nice” recoveries can keep loss grids manageable. Exact recovery conventions are desk‑ and model‑specific.

### 49.2.6 LHP Model: Closed-Form ETL Formulas

The Large Homogeneous Pool (LHP) model, developed by Vasicek and applied to CDOs by O'Kane and others, provides closed-form expressions for the loss distribution and ETL when the portfolio is large and homogeneous.

**Key assumption:** In the limit $N \to \infty$ with identical credits, the conditional loss distribution collapses to a point mass at the conditional expected loss:

$$L(T|Z) = (1-R) \cdot \Phi\left(\frac{C(T) - \beta Z}{\sqrt{1-\beta^2}}\right)$$

where $C(T) = \Phi^{-1}(q(T))$, $q(T)$ is the unconditional default probability, and $\beta$ is the asset correlation parameter (often written $\rho$ in some texts; $\beta = \sqrt{\rho}$).

**Unconditional loss distribution:** O'Kane derives in Chapter 16 that the cumulative distribution function of portfolio loss is:

$$\boxed{F(K) = \Pr(L(T) \leq K) = 1 - \Phi(A(K))}$$

where:
$$\boxed{A(K) = \frac{1}{\beta}\left(C(T) - \sqrt{1-\beta^2} \cdot \Phi^{-1}\left(\frac{K}{1-R}\right)\right)}$$

**ETL for equity tranche:** The expected loss on an equity tranche with detachment $K$ is:

$$\boxed{\psi(T, K) = E[\min(L(T), K)] = (1-R)\Phi_2\left(C(T), -A(K), -\beta\right) + K \cdot \Phi(A(K))}$$

where $\Phi_2(a, b, \rho)$ is the bivariate standard normal CDF with correlation $\rho$.

**Intuition:** The first term captures the expected loss when $L \lt K$ (partial losses), weighted by the bivariate probability. The second term captures full loss $K$ when the portfolio loss exceeds $K$, weighted by $\Phi(A(K)) = 1 - F(K)$.

**Practical usage:** The LHP formulas are extremely fast—requiring only evaluation of univariate and bivariate normal CDFs—making them ideal for:
- Quick pricing checks and sanity testing
- Sensitivity analysis requiring many recalculations
- Calibration inner loops where speed matters
- Building intuition for correlation effects

**Worked Example 49.2a (LHP ETL Calculation):**

Parameters: $q(T) = 5\%$ (5Y default probability), $R = 40\%$, $\beta = 0.3$ (correlation $\rho = 9\%$).

Tranche: $[3\%, 7\%]$ equity.

Step 1: $C(T) = \Phi^{-1}(0.05) = -1.645$

Step 2: For $K = 3\% = 0.03$:
$$A(0.03) = \frac{1}{0.3}\left(-1.645 - \sqrt{1-0.09} \cdot \Phi^{-1}(0.05)\right) = \frac{1}{0.3}(-1.645 - 0.954 \times (-1.645)) = \frac{-0.077}{0.3} = -0.257$$

Step 3: $\Phi(A(0.03)) = \Phi(-0.257) = 0.399$

Step 4: $\psi(T, 0.03) = 0.6 \times \Phi_2(-1.645, 0.257, -0.3) + 0.03 \times 0.399$

Using bivariate normal: $\Phi_2(-1.645, 0.257, -0.3) \approx 0.029$

$$\psi(T, 0.03) = 0.6 \times 0.029 + 0.03 \times 0.399 = 0.0174 + 0.0120 = 2.94\%$$

For $K = 7\%$: similar calculation yields $\psi(T, 0.07) \approx 3.0\%$ (nearly all portfolio loss falls below 7%).

ETL for $[3\%, 7\%]$: $\psi(0.07) - \psi(0.03) \approx 0.06\%$

This low ETL reflects that the 3-7% tranche is largely protected by the equity in this low-correlation environment.

### 49.2.7 Approximation Methods: Gaussian and Adjusted Binomial

O'Kane describes several computational approaches that trade accuracy for speed:

**Gaussian approximation (Shelton 2004):** Fit the conditional loss distribution with a Gaussian matching the first two moments:
$$\mu(Z) = \frac{1}{N} \sum_{i=1}^N p_i(T|Z)(1-R_i), \quad \sigma^2(Z) = \frac{1}{N^2} \sum_{i=1}^N p_i(T|Z)(1-p_i(T|Z))(1-R_i)^2$$

This yields a semi-closed-form conditional ETL that can be integrated analytically. O'Kane notes the conditional ETL for tranche $[K_1, K_2]$ is:

$$E[\min(L,K_2) - \min(L,K_1)|Z] = \sigma\left(\phi\left(\frac{\mu-K_1}{\sigma}\right) - \phi\left(\frac{\mu-K_2}{\sigma}\right)\right) + (\mu-K_1)\Phi\left(\frac{\mu-K_1}{\sigma}\right) - (\mu-K_2)\Phi\left(\frac{\mu-K_2}{\sigma}\right)$$

**Binomial approximation:** Use a single average conditional default probability $p(T|Z) = \frac{1}{N_c}\sum_i p_i(T|Z)$ and compute the loss distribution via simple binomial recursion. Faster than full recursion since complexity is $O(N_c)$ per factor value.

**Adjusted binomial approximation:** Adjust the binomial to match both the mean and variance of the exact multinomial distribution. O'Kane shows this involves scaling the binomial probabilities and adding/subtracting probability mass at specific points to match variance.

### 49.2.8 Accuracy Comparison of Methods

O'Kane provides detailed accuracy comparisons in Chapter 18.6. The following table summarizes typical pricing errors for CDX IG tranches (125 names, 5Y maturity):

| Method | 0-3% Error | 3-7% Error | 7-10% Error | 10-15% Error | 15-30% Error |
|:-------|:----------:|:----------:|:-----------:|:------------:|:------------:|
| **Gaussian approx** | ~56 bp | ~10 bp | ~5 bp | ~2 bp | <1 bp |
| **Binomial approx** | ~15 bp | ~5 bp | ~2 bp | ~1 bp | <1 bp |
| **Adjusted binomial** | <1 bp | <1 bp | <1 bp | <1 bp | <1 bp |
| **LHP (N→∞)** | ~5 bp | ~2 bp | ~1 bp | <1 bp | <1 bp |

**Key observations:**
1. **Gaussian approximation is least accurate for equity**—the loss distribution is skewed, not symmetric
2. **Adjusted binomial matches exact pricing to within 1 bp**—minimal extra cost versus standard binomial
3. **LHP errors scale as $O(1/N_c)$**—negligible for 125+ name portfolios
4. **All methods converge for senior tranches**—these depend only on tail probabilities, which all methods estimate similarly

> **Desk Reality: When to Use What**
>
> | Situation | Recommended Method |
> |-----------|-------------------|
> | Quick sanity check, intuition building | LHP closed-form |
> | Calibration inner loop (needs speed) | Gaussian or binomial approx |
> | Production pricing (accuracy required) | Full recursion or adjusted binomial |
> | Non-standard structure, complex payoff | Monte Carlo |
> | Bespoke tranche, CDO-squared | Monte Carlo (only option) |

### 49.2.9 Monte Carlo Methods for ETL

When analytical methods are insufficient—for non-standard tranches, CDO-squared structures, or complex dependencies—Monte Carlo simulation provides a flexible alternative.

**The Direct Approach (O'Kane Ch 14.7.1):**

1. Generate $P$ sets of $N_c$ correlated default times $\\{\tau_i^p\\}$
2. For each path $p$:
   - Remove default times beyond maturity $T$
   - Sort remaining defaults into ascending order
   - Compute cash flows on protection and premium legs
   - Discount to present value
3. Average across paths:

$$PV_{\text{prot}} = \frac{1}{P}\sum_{p=1}^P PV_{\text{prot},p}$$

$$\mathrm{RPV01} = \frac{1}{P}\sum_{p=1}^P \mathrm{RPV01}_p$$

**The Indirect Approach (O'Kane Ch 14.7.2):**

Instead of computing PVs directly, build the tranche survival curve:

1. Discretize $[0, T]$ into $N_T$ buckets; initialize $Q(k) = 1$ for all $k$
2. For each path $p$:
   - Compute tranche loss at each default time
   - Reduce $Q(k)$ by the loss fraction divided by $P$ where $k$ is the time bucket
3. Feed the survival curve $Q(k)$ to standard CDS pricing formulas

**Advantages of each approach:**

| Direct Approach | Indirect Approach |
|-----------------|-------------------|
| Handles any payoff structure | Faster: avoids discount factor calls in simulation |
| Required for complex products | Can price multiple maturities from one run |
| Straightforward implementation | Decouples model from product code |

**Variance reduction:** For senior tranches where losses are rare, naive Monte Carlo converges slowly. Techniques include:
- Importance sampling (oversample bad economic states)
- Control variates (use LHP price as control)
- Stratified sampling on the factor $Z$

**Typical path counts:** O'Kane reports using 100,000 paths for standard tranches. For CDO-squared with Monte Carlo integration over sub-tranches, computational cost increases significantly.

**Standard error of breakeven spread:** O'Kane shows in Chapter 14.8 that the standard error of the spread estimate is approximately:

$$SE(s) \approx \frac{SE(PV_{\text{prot}})}{\mathrm{RPV01}}$$

This means spread estimation precision depends on the RPV01—senior tranches with high RPV01 have more precise spread estimates for the same number of paths.

### 49.2.10 The Conservation of Expected Loss

A fundamental arbitrage constraint relates tranche ETLs to the portfolio expected loss. O'Kane proves in Section 12.7.1 that if we buy protection on contiguous tranches covering the entire capital structure $[0, K_1], [K_1, K_2], \ldots, [K_{M-1}, 1]$:

$$\text{Total expected loss} = \sum_{m=1}^M (K_m - K_{m-1}) \cdot E[L(T, K_{m-1}, K_m)] = E[L(T)]$$

The sum of the tranche expected losses (weighted by width) equals the portfolio expected loss. This conservation property must hold for any valid pricing model—it's a no-arbitrage constraint.

Desk interpretation: if you buy protection on **all** contiguous tranches (covering the full loss range), you have bought protection on the whole portfolio loss process. The protection legs therefore add up consistently across the capital structure (ignoring premium‑timing differences).

---

## 49.3 Protection Leg PV: Discounted Expected Loss Increments

### 49.3.1 Continuous-Time Formulation

The protection leg pays realized tranche losses when they occur. O'Kane writes the protection leg PV (for $USD 1$ face value) as:

$$PV_{\text{prot}} = \int_0^T Z(s) \\, E[dL_{\text{tr}}(s)]$$

where $dL_{\text{tr}}(s)$ is the increment of normalized tranche loss at time $s$. Equivalently, in terms of the tranche survival curve:

$$PV_{\text{prot}} = \int_0^T Z(s) (-dQ(s))$$

This is exactly analogous to the CDS protection leg with zero recovery—the "zero recovery" comes from the fact that tranche losses, once realized, are not recovered within the tranche structure.

### 49.3.2 Discrete-Time Approximation

For practical computation on a grid $0 = t_0 \lt t_1 \lt \cdots \lt t_N = T$:

$$\boxed{PV_{\text{prot}}(USD ) = N_{\text{port}} \sum_{i=1}^N Z(t_i) \left(\mathrm{ETL}(t_i) - \mathrm{ETL}(t_{i-1})\right)}$$

Each term $\Delta\mathrm{ETL}_i = \mathrm{ETL}(t_i) - \mathrm{ETL}(t_{i-1})$ represents the expected loss "arriving" in interval $(t_{i-1}, t_i]$, discounted to today.

**Check (sign + monotonicity):** ETL should be non-decreasing in $t$ for a fixed tranche $[A,D]$, so $\Delta\mathrm{ETL}_i \ge 0$. If your computed ETL decreases with time, you have an input/model/implementation inconsistency and the protection leg PV formula can produce nonsensical negative contributions.

O'Kane improves accuracy using the average of bounds:

$$PV_{\text{prot}} \approx \frac{1}{2} \sum_{i=1}^N (Z(t_{i-1}) + Z(t_i)) \cdot \Delta\mathrm{ETL}_i \cdot N_{\text{port}}$$

This averages the discount factors at the interval endpoints, reducing discretization error. In practice it can materially improve accuracy for a given time grid.

**Worked Example 49.3 (Protection Leg PV):**

Tranche $[3\%, 7\%]$, portfolio notional `N_port = USD 100mm`. ETL curve and discount factors:

| Time | ETL | Delta ETL | $Z(t)$ | $Z \times$ Delta ETL |
|:----:|:---:|:-----------:|:------:|:--------------------:|
| 0 | 0.0% | — | 1.000 | — |
| 1 | 0.2% | 0.2% | 0.970 | 0.00194 |
| 3 | 0.8% | 0.6% | 0.900 | 0.00540 |
| 5 | 1.2% | 0.4% | 0.820 | 0.00328 |

$$PV_{\text{prot}} = (0.00194 + 0.00540 + 0.00328) \times USD 100\text{mm} = USD 1.062\text{mm}$$

---

## 49.4 Premium Leg PV: The Average-Notional Approximation

### 49.4.1 Why an Approximation is Needed

The premium leg pays spread $s$ on the outstanding tranche notional. The challenge is that losses can occur at any time within a coupon period, not just at the endpoints. If a credit defaults mid-quarter, the tranche notional shrinks immediately, but the premium payment at quarter-end should reflect this reduced notional.

A common and useful approximation is to assume the spread is paid on the **average tranche notional** over the coupon period (average of start and end notionals). This captures, in a simple way, the fact that losses can occur inside the period.

### 49.4.2 The Average-Notional Formula

The premium leg PV (per $USD 1$ of tranche face value) using the average-notional approximation is:

$$PV_{\text{prem}}(USD 1\text{ face}) = s \sum_{i=1}^N \alpha_i Z(t_i) \\, E\left[1 - \frac{L_{\text{tr}}(t_{i-1}) + L_{\text{tr}}(t_i)}{2}\right]$$

Equivalently, using the tranche survival $Q(t)$:

$$\boxed{PV_{\text{prem}}(USD 1\text{ face}) = \frac{s}{2} \sum_{i=1}^N \alpha_i Z(t_i) \left(Q(t_{i-1}) + Q(t_i)\right)}$$

**Check (limiting cases):**
- If the tranche is expected to survive with essentially no losses over the horizon ($Q(t)\approx 1$), then the premium leg reduces to $s$ times a near risk-free annuity (the usual “DV01-style” intuition).
- If the tranche is expected to be rapidly written down ($Q(t)$ drops quickly toward 0), then the premium base amortizes and premium PV becomes small even if the contractual spread $s$ is very large.

In portfolio-notional dollars:

$$\boxed{PV_{\text{prem}}(USD ) = s \\, N_{\text{port}} \sum_{i=1}^N \alpha_i Z(t_i) \\, E\left[\frac{\text{ON}(t_{i-1}) + \text{ON}(t_i)}{2}\right]}$$

### 49.4.3 Accuracy of the Approximation

This mid-period approximation is generally accurate when coupon periods are short (e.g., quarterly) and losses don’t arrive in highly concentrated bursts. It can be less accurate in distressed scenarios with clustered defaults.

**When the approximation may be less accurate:**
- Very distressed portfolios where multiple defaults cluster in time
- Long coupon periods (e.g., annual)
- Equity tranches with high probability of rapid loss accumulation

For these cases, more sophisticated approaches (Monte Carlo with exact timing) may be warranted.

### 49.4.4 Premium Accrued at Default

As with CDS, if the tranche experiences losses within a coupon period, the protection seller may owe accrued premium on the lost notional. O'Kane's formula implicitly handles this through the survival curve machinery—the same approach used in Section 6.5 for CDS premium legs.

**Worked Example 49.4 (Premium Leg PV):**

Same tranche and curve as Example 49.3. Spread $s = 500$ bp/year = 0.05. Accruals: `alpha1 = 1y`, `alpha2 = 2y`, `alpha3 = 2y`. EON from ETL: $\text{EON}(t) = 4\% - \mathrm{ETL}(t)$.

| Interval | Avg EON | $\alpha$ | $Z$ | $Z \times \alpha \times \text{Avg EON}$ |
|:--------:|:-------:|:--------:|:---:|:---------------------------------------:|
| [0,1] | (4.0%+3.8%)/2 = 3.9% | 1 | 0.97 | 0.03783 |
| [1,3] | (3.8%+3.2%)/2 = 3.5% | 2 | 0.90 | 0.06300 |
| [3,5] | (3.2%+2.8%)/2 = 3.0% | 2 | 0.82 | 0.04920 |

$$PV01_{\text{prem}} = (0.03783 + 0.06300 + 0.04920) \times USD 100\text{mm} = USD 15.003\text{mm}$$

$$PV_{\text{prem}} = 0.05 \times USD 15.003\text{mm} = USD 0.750\text{mm}$$

---

## 49.5 Tranche RPV01, Par Spread, Mark-to-Market, and Upfront

### 49.5.1 Tranche Risky PV01

As with a CDS contract, we can define a $T$-maturity tranche risky PV01 (RPV01) as the present value of $USD 1$ paid on the premium leg of the tranche. Per $USD 1$ of tranche face notional, at time zero we have (average‑notional approximation):

$$\boxed{\mathrm{RPV01}(0) = \frac{1}{2} \sum_{n=1}^{N_T} \Delta(t_{n-1}, t_n) Z(t_n) \left(Q(t_{n-1}, K_1, K_2) + Q(t_n, K_1, K_2)\right)}$$

where $Z(t_n)$ is the discount factor and $Q(t, K_1, K_2)$ is the tranche survival probability. In O'Kane's notation, $Q(t_n,K_1,K_2)=\mathbb{E}[1-L(t_n,K_1,K_2)]$, where $L(t_n,K_1,K_2)$ is the tranche cumulative loss fraction (as a fraction of tranche notional).

Interpretation: tranche RPV01 behaves like CDS RPV01. For a very safe tranche, expected notional stays close to full, so RPV01 is near a risk‑free PV01. For a riskier tranche, expected notional amortizes, so RPV01 is materially smaller.

**Units and desk usage:**

- The formula above returns **years per USD 1 tranche face**.
- To convert to dollars on a position, multiply by tranche face notional $N_{\text{tr}}^{\text{face}} = W \cdot N_{\text{port}}$.
- Traders often quote “RPV01” in **dollars per 1 bp**:
  $$RPV01bp = 10^{-4} \times NtrFace \times RPV01$$
- **Bump definition (spread risk):**
  - Bump object: the **running spread** $s$ on the premium leg, holding the ETL/$Q$ term structure and discount factors fixed.
  - Bump size: $1\text{ bp} = 10^{-4}$ (in decimal per year).
  - Units: currency per 1 bp, for the stated tranche face notional.
  - Sign (protection buyer): `Delta PV ≈ -RPV01bp × Delta s(bp)`.

**Comparison to single-name CDS RPV01:** The tranche RPV01 has the same mathematical structure as the CDS RPV01 in Chapter 41, but with the issuer survival curve replaced by the tranche survival curve. This reinforces the CDS-tranche analogy: once you have the survival curve, pricing follows the same formulas.

### 49.5.2 Par Spread Derivation

The par spread $s^{\star}$ equates the protection leg PV to the premium leg PV at initiation:

$$PV_{\text{prot}} = s^{\star} \times PV01_{\text{prem}}$$

Solving:

$$\boxed{s^{\star} = \frac{PV_{\text{prot}}}{PV01_{\text{prem}}} = \frac{\sum_i Z(t_i) \Delta\mathrm{ETL}_i}{\sum_i \alpha_i Z(t_i) \frac{Q(t_{i-1}) + Q(t_i)}{2}}}$$

**Units check:**
- Numerator: unitless (discount factor × ETL increment as fraction)
- Denominator: years × unitless × unitless = years
- Result: per year (multiply by 10,000 for bp/year)

**Worked Example 49.5 (Par Spread Calculation):**

From Examples 49.3–49.4:
- $PV_{\text{prot}} = USD 1.062\text{mm}$
- $PV01_{\text{prem}} = USD 15.003\text{mm}$

$$s^{\star} = \frac{USD 1.062\text{mm}}{USD 15.003\text{mm}} = 0.0708 = 708 \text{ bp/year}$$

### 49.5.3 Mark-to-Market Value

The par spread is the entry price for a new trade. But what about existing positions where the contractual spread $s_0$ differs from the current par spread $s^{\star}$?

**Tranche value formula:** For a protection buyer with contractual spread $s_0$:

$$\boxed{V = PV_{\text{prot}} - s_0 \times PV01_{\text{prem}}}$$

Equivalently:

$$\boxed{V = (s^{\star} - s_0) \times PV01_{\text{prem}}}$$

**Sign conventions:**
- Protection **buyer** has $V \gt 0$ when spreads **widen** (par spread rises above contractual)
- Protection **seller** has $V \gt 0$ when spreads **tighten**

**Worked Example 49.6 (Mark-to-Market):**

A trader sold protection on the 3-7% tranche at 500 bp (contractual spread). Today's par spread is 708 bp, and $PV01_{\text{prem}}=USD 15.003\text{mm}$ for this position.

Value to protection seller:
$$V = (s_0 - s^{\star}) \times PV01_{\text{prem}} = (0.05 - 0.0708) \times USD 15.003\text{mm} \approx -USD 0.31\text{mm}$$

The position is underwater: spreads widened, so the protection seller owes more in expected losses than they receive in premium.

> **Desk Reality: P&L Decomposition**
>
> When spreads move, tranche P&L has two components:
> 1. **Spread P&L:** `(new par spread - old par spread) × PV01_prem`
> 2. **Time decay / carry:** Premium received minus expected loss accrued
>
> Additionally, $PV01_{\text{prem}}$ (and $RPV01$) changes as survival probabilities evolve, creating **convexity P&L** (second-order effect).
>
> Middle-office reconciliation tip: If the P&L attribution doesn't tie out, check whether the system is using constant RPV01 versus updated RPV01. The difference is the "gamma" effect.

### 49.5.4 Upfront Payment Conventions for Equity Tranches

Some tranche markets standardize equity-tranche quotes as a **fixed running coupon** plus an **upfront payment** quoted as a percent of tranche notional (often called “points upfront”). A common description for index equity tranches is:

> “The equity tranche of the iTraxx Index is quoted as an upfront percentage of the notional plus 500 basis point running-fee paid quarterly.”

Tranche quotes can be in basis points or involve an upfront payment. In the latter case, the spread in basis points is fixed and the upfront payment, as a percent of the tranche principal, is either input or implied (e.g., a fixed 500 bp running fee for standard equity index tranches).

One way to remember the quote algebra is:
$$
Upoints = ELexp - \frac{s \times PVpmts}{10{,}000}
$$
where $\text{ExpLoss}$ is expected loss as a percent of tranche principal and $\text{PVPmts}$ is the present value of expected premium payments computed at 10,000 bp/year.

Define $V_{\text{clean}}$ as the present value to the **protection buyer** of the contractual legs *excluding* any upfront (i.e., protection PV minus fixed-coupon premium PV):

$$\boxed{V_{\text{clean}} = PV_{\text{prot}} - s_{\text{fixed}} \times PV01_{\text{prem}}}$$

In this chapter, quote the upfront cash amount $U$ as a **positive number when it is paid by the protection buyer** to the protection seller at settlement. At trade inception, the quote $(s_{\text{fixed}}, U)$ is chosen so the net PV is zero:

$$\boxed{0 = V_{\text{clean}} - U \quad\Rightarrow\quad U = V_{\text{clean}}}$$

Dividing by tranche face notional $N_{\text{tr}}^{\text{face}}$ gives “points upfront” (a fraction of tranche notional). Using $PV_{\text{prot}} = s^{\star} \times PV01_{\text{prem}}$ and $RPV01 = PV01_{\text{prem}}/N_{\text{tr}}^{\text{face}}$, this becomes:

$$\boxed{Upoints = (s^{\star} - s_{\text{fixed}}) \times RPV01}$$

Sign: if $s^{\star} \gt s_{\text{fixed}}$, the protection buyer is paying “too little” running premium, so **the protection buyer pays positive upfront**.

> **Pitfall — Quote format and bp scaling:** Equity tranche quotes are easy to misread.
> **Why it matters:** You can be off by 100× (percent vs decimal) or 10,000× (bp vs decimal), and you can invert “price” vs “points upfront”.
> **Quick check:** Compute both $V_{\text{clean}}=PV_{\text{prot}}-s_{\text{fixed}}PV01$ (currency) and `Upoints = V_clean / N_tr_face` (percent of tranche notional). They must match the quote format you’re using.

**Worked Example 49.7 (Equity tranche quote: 500 bp running + upfront, with dates)**

**Context**
- You are quoting an equity tranche as “500 bp running + upfront points” and want to compute the upfront from a model ETL curve.

**Timeline (example assumptions)**
- Trade/valuation date: 2026-03-20
- Accrual start: 2026-03-20
- Premium payment dates: 2026-06-20, 2026-09-20, 2026-12-20, 2027-03-20
- Maturity: 2027-03-20

**Inputs**
- Tranche: $A=0\%$, $D=3\%$, width $W=3\%$
- Portfolio notional: $N_{\text{port}} = USD 1{,}000{,}000{,}000$ so tranche face notional $N_{\text{tr}}^{\text{face}} = W \cdot N_{\text{port}} = USD 30{,}000{,}000$
- Fixed running coupon: $s_{\text{fixed}} = 500$ bp/year $=0.05$
- Day count: ACT/360; assume $\delta=0.25$ each quarter (to keep arithmetic short)
- Discount factors $Z(t)$ and ETL curve (cumulative ETL as a % of portfolio notional):

| Date | $Z(t)$ | $\mathrm{ETL}(t)$ |
|---|---:|---:|
| 2026-06-20 | 0.9925 | 0.10% |
| 2026-09-20 | 0.9850 | 0.18% |
| 2026-12-20 | 0.9775 | 0.28% |
| 2027-03-20 | 0.9700 | 0.40% |

**Outputs (what you produce)**
- $PV_{\text{prot}}$, $PV01_{\text{prem}}$, par spread $s^{\star}$
- Upfront (points) and upfront (cash) for the quote `(500 bp, U_points)`

**Step-by-step**
1. Compute protection-leg PV from ETL increments:
   $PV_{\text{prot}} = N_{\text{port}} \sum_i Z(t_i)\,\Delta ETL(i)$ with `Delta ETL(i) = ETL(current date) - ETL(previous date)`.

| Date | Delta ETL | Expected protection cashflow $=N_{\text{port}} \times \Delta ETL$ | PV contribution $=Z \cdot N_{\text{port}} \times \Delta ETL$ |
|---|---:|---:|---:|
| 2026-06-20 | 0.10% | USD 1.000mm | USD 0.992mm |
| 2026-09-20 | 0.08% | USD 0.800mm | USD 0.788mm |
| 2026-12-20 | 0.10% | USD 1.000mm | USD 0.978mm |
| 2027-03-20 | 0.12% | USD 1.200mm | USD 1.164mm |

Result: $PV_{\text{prot}} \approx 3.922$ mm USD.

2. Compute premium-leg PV01 from expected outstanding notional:
   $EON(t)=W-ETL(t)$ and $PV01_{\text{prem}} = N_{\text{port}} \sum_i \delta\, Z(t_i)\,\frac{EON(t_{i-1})+EON(t_i)}{2}$.

| Date | $\text{EON}(t)$ | Avg EON over period | Premium (at 500 bp) $=s_{\text{fixed}}\delta N_{\text{port}}\text{AvgEON}$ |
|---|---:|---:|---:|
| 2026-06-20 | 2.90% | 2.95% | USD 0.369mm |
| 2026-09-20 | 2.82% | 2.86% | USD 0.358mm |
| 2026-12-20 | 2.72% | 2.77% | USD 0.346mm |
| 2027-03-20 | 2.60% | 2.66% | USD 0.333mm |

$PV01_{\text{prem}} \approx 27.582$ mm USD and $PV_{\text{prem}}(500\text{ bp}) = 0.05 \times PV01_{\text{prem}} \approx 1.379$ mm USD.

3. Compute par spread and upfront: $s^{\star}=\frac{PV_{\text{prot}}}{PV01_{\text{prem}}} \approx \frac{3.922}{27.582}=0.1422=1422\text{ bp/year}$.

Clean value (protection buyer, excluding upfront): $V_{\text{clean}} = PV_{\text{prot}} - 0.05 \times PV01_{\text{prem}} \approx 2.543$ mm USD.

Therefore the quote “500 bp running + upfront” implies an upfront **paid by the protection buyer** of about USD 2.543mm cash, which is $\frac{2.543}{30.0} \approx 8.48\%$ in points.

**P&L / Risk interpretation**
- This trade is **long default/loss risk** for the protection buyer: higher ETL increases $PV_{\text{prot}}$, widens $s^{\star}$, and increases upfront.
- The running-spread risk is summarized by `RPV01`: a +1 bp increase in the running spread (holding ETL fixed) changes PV by approximately $-\mathrm{RPV01}_{\text{bp}}$ for the protection buyer.

**Sanity checks**
- Units check: ETL and EON are fractions of **portfolio** notional; multiply by $N_{\text{port}}$ to get currency.
- Sign check: if ETL is identically 0, then $PV_{\text{prot}}=0\Rightarrow s^{\star}=0\Rightarrow$ upfront $=0$.
- Reproduction check: the two 4-row tables above are a spreadsheet implementation of the pricing formulas.

**Debug checklist**
- Are you mixing **portfolio notional** vs **tranche notional** (divide/multiply by $W$ correctly)?
- Did you convert bp to decimal correctly ($500\text{ bp}=0.05$, $1\text{ bp}=10^{-4}$)?
- Are you using end-of-period outstanding notional or average-notional (and is your counterparty using the same approximation)?
- Did you confirm whether the quote is “points upfront” or “price”?

> **Desk Reality:** Confirm the quote format before you reconcile P&L.
>
> Always confirm *both* the fixed coupon and the quoting style: “Is that **price** or **points upfront**? What’s the **running coupon**? What’s the **day count / pay frequency**?”

### 49.5.5 Spread-ETL Convexity

The par spread is approximately the ratio of total expected loss to total expected outstanding duration. When ETL increases:
- Numerator increases (more expected protection payments)
- Denominator typically decreases (lower expected outstanding)

Both effects push the par spread higher, creating **convexity** in the spread-ETL relationship.

**Worked Example 49.8 (Spread Sensitivity to ETL):**

Using our base case, suppose ETL at 5Y increases from 1.2% to 1.5% (a 0.3% increase). The incremental protection PV is approximately $0.82 \times 0.3\% \times USD 100\text{mm} = USD 0.246\text{mm}$.

The EON decreases by 0.3%, reducing PV01 by approximately $0.82 \times 2 \times 0.15\% \times USD 100\text{mm} = USD 0.246\text{mm}$ (rough estimate).

New par spread: $\frac{USD 1.062 + USD 0.246}{USD 15.003 - USD 0.246} \approx \frac{USD 1.308}{USD 14.757} = 886$ bp/year.

A 0.3% increase in terminal ETL raises the spread by ~180 bp—demonstrating the leverage inherent in tranche pricing.

---

## 49.6 Term Structure of ETL

### 49.6.1 ETL Across Maturities

The ETL is a function of both strike and time. O'Kane's notation $\psi(T, K)$ emphasizes this two-dimensional surface. Key properties:

1. **Monotonicity in time:** $\psi(T_1, K) \leq \psi(T_2, K)$ for $T_1 \lt T_2$. Losses cannot reverse, so ETL is non-decreasing in maturity.

2. **Time-derivative constraint:** The probability of loss exceeding $K$ must also be non-decreasing, implying:
   $$\frac{\partial^2 \psi(T, K)}{\partial T \partial K} \geq 0$$

3. **Concavity in strike:** $\frac{\partial^2 \psi(T, K)}{\partial K^2} \leq 0$. This follows from the requirement that the implied loss density be non-negative.

### 49.6.2 Evolution from 1Y to 5Y

**Worked Example 49.9 (Term Structure of ETL):**

Consider how ETL evolves for different tranches as we move from 1Y to 5Y horizon:

| Horizon | 0–3% ETL | 3–7% ETL | 7–10% ETL | Portfolio EL |
|:-------:|:--------:|:--------:|:---------:|:------------:|
| 1Y | 0.5% | 0.1% | 0.01% | 0.61% |
| 2Y | 1.2% | 0.3% | 0.05% | 1.55% |
| 3Y | 1.8% | 0.6% | 0.12% | 2.52% |
| 5Y | 2.8% | 1.2% | 0.35% | 4.35% |

Observations:
- **Equity ETL** grows quickly, approaching tranche width (3%) by 5Y
- **Mezzanine ETL** starts near zero, accelerates as portfolio losses reach attachment
- **Senior ETL** remains small but grows exponentially in relative terms
- **Conservation check:** Sum of notional-weighted ETLs should equal portfolio EL

This term structure drives the shape of the tranche survival curve and hence all pricing.

---

## 49.7 No-Arbitrage Constraints on the ETL Surface

O'Kane's Chapter 19 develops rigorous no-arbitrage conditions for the ETL surface. These constraints are essential for model validation and interpolation.

### 49.7.1 Boundary Conditions

1. **Zero width implies zero loss:** $\psi(T, 0) = 0$
2. **Zero maturity implies zero loss:** $\psi(0, K) = 0$
3. **ETL cannot exceed width:** $\psi(T, K) \leq K$
4. **Full portfolio coverage:** $\psi(T, K) = E[L(T)]$ for $K \geq L_{\max}$

### 49.7.2 Shape Constraints

**Concavity:** The ETL curve must be concave in $K$:
$$\frac{\partial^2 \psi(T, K)}{\partial K^2} = -f(K) \leq 0$$

where $f(K)$ is the implied loss density. Positive density requires non-positive second derivative.

**Monotonicity:** ETL increases in both $T$ and $K$:
$$\frac{\partial \psi}{\partial T} \geq 0, \quad \frac{\partial \psi}{\partial K} \geq 0$$

**Cross-derivative:** The slope in $K$ must be non-decreasing in $T$:
$$\frac{\partial^2 \psi(T, K)}{\partial T \partial K} \geq 0$$

These constraints define the envelope of valid ETL surfaces and can be used to assess model arbitrage.

> **Visual Intuition: The ETL Curve Shape**
>
> Think of the ETL curve $\psi(T, K)$ as a function of strike $K$ for fixed $T$. It starts at zero for $K=0$, rises quickly at first (equity losses are expected), then flattens out as you approach the senior strikes (losses rarely reach there). The curve "bends downward like a smile turned upside down"—this is the concavity requirement.
>
> If the curve were convex (bending upward), you could create arbitrage by buying senior protection and selling junior protection in certain proportions.

---

## 49.8 Implementation Notes

### 49.8.1 Efficient ETL Computation

**For discrete loss distributions:**
1. Build the loss distribution at each coupon date (using recursion, FFT, or simulation)
2. Compute $\text{TL}(L_j)$ for each loss scenario $L_j$
3. Weight by probabilities: $\mathrm{ETL} = \sum_j p_j \cdot \text{TL}(L_j)$

**For LHP/Gaussian approximations:**
1. Compute conditional ETL analytically for each factor value $Z$
2. Integrate over $Z$ using Gaussian quadrature or trapezium rule
3. O'Kane recommends $N_Z = 50$ factor grid points for typical accuracy

**For Monte Carlo:**
1. Simulate portfolio loss paths
2. Compute tranche loss for each path
3. Average across paths (with variance reduction as needed)

### 49.8.2 ETL Interpolation Between Coupon Dates

For protection leg PV computation, we need ETL at each payment date. Common approaches:
- **Linear interpolation:** Fast but may violate monotonicity constraints near high-loss scenarios
- **Piecewise constant forward rate:** Assume constant hazard rate between calibration points
- **Spline interpolation:** Smoother but must enforce concavity constraints

### 49.8.3 Common Calculation Errors

1. **Unit confusion:** Mixing portfolio fractions with tranche fractions or dollar amounts. Always track units explicitly.

2. **Ignoring the cap:** Forgetting that tranche loss is capped at $W$, leading to impossible ETL values.

3. **Double-counting:** Adding equity and mezzanine ETL without accounting for overlap (the correct decomposition uses `psi(D) - psi(A)`).

4. **Conservation violation:** If sum of ETLs across tranches doesn't equal portfolio EL, there's a modeling or calculation error.

5. **Negative tranche survival:** If $Q(t) \lt 0$, the ETL exceeds the tranche width—impossible.

### 49.8.4 Implementation Validation Checklist

Before trusting a new ETL implementation, run these validation tests:

| Test | What It Checks | How to Perform |
|------|----------------|----------------|
| **Conservation** | Sum of ETLs = Portfolio EL | Sum tranche ETLs weighted by width; compare to $E[L(T)]$ |
| **Monotonicity in T** | ETL increases with maturity | ETL(1Y) ≤ ETL(2Y) ≤ ETL(5Y) for same tranche |
| **Monotonicity in K** | ETL increases with strike | ETL(0-3%) ≤ ETL(0-7%) ≤ ETL(0-10%) |
| **Boundary at zero** | ETL(0, K) = 0 | At t=0, no losses have occurred |
| **Cap check** | ETL ≤ tranche width | ETL for [3%, 7%] must be ≤ 4% |
| **Concavity** | Second derivative in K ≤ 0 | $\psi(0.07) - 2\psi(0.05) + \psi(0.03) \leq 0$ |
| **LHP cross-check** | Compare to closed-form | For homogeneous portfolio, exact should match LHP to $O(1/N_c)$ |

**Worked Example 49.10 (Detecting an Error via Conservation):**

Suppose your model produces:

| Tranche | Width | ETL | Contribution |
|:-------:|:-----:|:---:|:------------:|
| 0-3% | 3% | 1.3% | 1.3% |
| 3-7% | 4% | 1.0% | 1.0% |
| 7-10% | 3% | 0.4% | 0.4% |
| 10-100% | 90% | 0.3% | **0.3%** |

Total = 3.0%, but portfolio expected loss = 3.5%

**Diagnosis:** The 0.5% gap suggests the senior tranche ETL is underestimated. Common cause: truncating the loss distribution before the senior attachment point, missing tail probability.

---

## 49.9 Worked Examples: Complete Calculations

### Example 49.11: Comparing Attachment Levels

Discrete horizon distribution: $L = \\{0\%, 2\%, 6\%, 15\%\\}$ with $p = \\{0.5, 0.2, 0.2, 0.1\\}$.

$T = 5$, $Z(T) = 0.82$, $\alpha = 5$, average-outstanding approximation.

**Equity tranche $[0\%, 3\%]$, $W = 3\%$:**

| $L$ | TL | $p$ | $p \times$ TL |
|:---:|:--:|:---:|:-------------:|
| 0% | 0 | 0.5 | 0 |
| 2% | 2% | 0.2 | 0.4% |
| 6% | 3% | 0.2 | 0.6% |
| 15% | 3% | 0.1 | 0.3% |

$\mathrm{ETL} = 1.3\%$, $\text{EON} = 1.7\%$

$PV_{\text{prot}} = 0.82 \times 1.3\% \times USD 100\text{mm} = USD 1.066\text{mm}$

$PV01 = 0.82 \times 5 \times \frac{0.03 + 0.017}{2} \times USD 100\text{mm} = USD 9.64\text{mm}$

$s^{\star} = USD 1.066/USD 9.64 = 11.06\% = 1106$ bp/year

**Mezzanine tranche $[3\%, 7\%]$, $W = 4\%$:**

| $L$ | TL | $p$ | $p \times$ TL |
|:---:|:--:|:---:|:-------------:|
| 0% | 0 | 0.5 | 0 |
| 2% | 0 | 0.2 | 0 |
| 6% | 3% | 0.2 | 0.6% |
| 15% | 4% | 0.1 | 0.4% |

$\mathrm{ETL} = 1.0\%$, $\text{EON} = 3.0\%$

$PV_{\text{prot}} = 0.82 \times 1.0\% \times USD 100\text{mm} = USD 0.820\text{mm}$

$PV01 = 0.82 \times 5 \times \frac{0.04 + 0.03}{2} \times USD 100\text{mm} = USD 14.35\text{mm}$

$s^{\star} = USD 0.820/USD 14.35 = 5.71\% = 571$ bp/year

**Interpretation:** Equity trades wider (1106 bp vs 571 bp) because it absorbs losses first. The mezz has higher EON (3% vs 1.7%) providing more premium cushion.

### Example 49.12: Tail Sensitivity

Same distribution but shift probability mass to the tail:

**Baseline:** $p = \\{0.5, 0.2, 0.2, 0.1\\}$

**Tail-heavy:** $p = \\{0.4, 0.2, 0.1, 0.3\\}$

**Senior tranche $[10\%, 15\%]$, $W = 5\%$:**

Only the 15% scenario hits this tranche (TL = 5% = full wipeout).

| Distribution | ETL | EON | $PV_{\text{prot}}$ | $PV01$ | $s^{\star}$ |
|:------------:|:---:|:---:|:------------------:|:------:|:-----:|
| Baseline | 0.5% | 4.5% | USD 0.41mm | USD 19.5mm | 211 bp |
| Tail-heavy | 1.5% | 3.5% | USD 1.23mm | USD 17.4mm | 707 bp |

The senior spread triples (211 → 707 bp) because its ETL depends entirely on the extreme tail probability, which tripled from 10% to 30%.

### Example 49.13: Recovery Sensitivity

Higher recoveries reduce loss severity. If losses scale by 5/6 (recoveries up 10%):

**Original losses:** $\\{0\%, 2\%, 6\%, 15\%\\}$
**Scaled losses:** $\\{0\%, 1.67\%, 5\%, 12.5\%\\}$

**Tranche $[3\%, 7\%]$:**

| Scaled $L$ | TL | $p$ | $p \times$ TL |
|:----------:|:--:|:---:|:-------------:|
| 0% | 0 | 0.5 | 0 |
| 1.67% | 0 | 0.2 | 0 |
| 5% | 2% | 0.2 | 0.4% |
| 12.5% | 4% | 0.1 | 0.4% |

$\mathrm{ETL} = 0.8\%$, $\text{EON} = 3.2\%$

$s^{\star} = \frac{0.82 \times 0.008}{0.82 \times 5 \times 0.036} = 444$ bp/year

vs 571 bp baseline—higher recovery reduces the spread.

### Example 49.14: Multi-Period ETL Curve

Building on Example 49.3, here's the full protection leg computation with quarterly detail:

| Quarter | $t$ | ETL | Delta ETL | $Z(t)$ | $Z \times$ Delta ETL |
|:-------:|:---:|:---:|:-----------:|:------:|:--------------------:|
| 1 | 0.25 | 0.05% | 0.05% | 0.988 | 0.000494 |
| 2 | 0.50 | 0.10% | 0.05% | 0.976 | 0.000488 |
| 3 | 0.75 | 0.15% | 0.05% | 0.964 | 0.000482 |
| 4 | 1.00 | 0.20% | 0.05% | 0.952 | 0.000476 |
| ... | ... | ... | ... | ... | ... |
| 20 | 5.00 | 1.20% | 0.05% | 0.820 | 0.000410 |

Total protection PV fraction ≈ 0.0096, times USD 100mm = USD 0.96mm.

### Example 49.15: Conservation Check

Verify that tranche ETLs sum to portfolio expected loss:

| Tranche | Width | ETL | Width × ETL/Width = Contribution |
|:-------:|:-----:|:---:|:--------------------------------:|
| 0–3% | 3% | 1.3% | 1.3% |
| 3–7% | 4% | 1.0% | 1.0% |
| 7–10% | 3% | 0.4% | 0.4% |
| 10–15% | 5% | 0.5% | 0.5% |
| 15–60% | 45% | 0.3% | 0.3% |

Total = 3.5% = Portfolio expected loss ✓

---

## Summary

1. **Portfolio loss $L(t)$** is the single state variable; tranche loss is a deterministic nonlinear function of it.

2. **ETL** = $E[\text{TL}(L(t))]$ = $E[\min(L,D) - \min(L,A)]$ is the central pricing object.

3. **For equity tranches**, ETL equals the base tranche loss $\psi(T,K) = E[\min(L(T),K)]$.

4. **Any tranche's ETL** equals the difference of two equity ETLs: $\psi(D) - \psi(A)$.

5. **Protection leg PV** = discounted expected loss increments = $\sum Z_i \Delta\mathrm{ETL}_i$.

6. **Premium leg PV** uses the average-notional approximation: spread × discounted expected outstanding.

7. **Tranche RPV01** parallels CDS RPV01 with the tranche survival curve replacing issuer survival.

8. **Par spread** = $PV_{\text{prot}} / PV01_{\text{prem}}$—higher ETL and lower outstanding both widen spreads.

9. **Mark-to-market value** = $(s^{\star} - s_0) \times PV01_{\text{prem}}$ for the protection buyer (equivalently $(s^{\star} - s_0)\times \mathrm{RPV01}\times N_{\text{tr}}^{\text{face}}$); use upfront convention for equity.

10. **Conservation of expected loss** across the capital structure is a no-arbitrage constraint.

11. **ETL surface** must be monotonic, concave in strike, with non-negative implied density.

12. **Computational methods** range from fast approximations (LHP, Gaussian) to exact recursion to Monte Carlo.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Portfolio Loss $L(t)$ | Fraction of portfolio lost to defaults by time $t$ | The single state variable for tranche pricing |
| Tranche Loss $\text{TL}(L)$ | $\min(\max(L-A,0), W)$ | Nonlinear payoff creating leverage across capital structure |
| ETL | $E[\text{TL}(L(t))]$ | Drives both legs of tranche PV |
| Tranche Survival $Q(t)$ | $E[1 - L_{\text{tr}}(t)] = \text{EON}/W$ | Enables CDS pricing analogy |
| RPV01 | PV of paying 1/year on premium leg | Measures tranche duration; converts spread to value |
| Par Spread $s^{\star}$ | $PV_{\text{prot}}/PV01_{\text{prem}}$ | Fair running spread at inception |
| MTM Value | $(s^{\star} - s_0)\times PV01_{\text{prem}}$ | Value of an existing position |
| Upfront (points) | $(s^{\star} - s_{\text{fixed}})\times \mathrm{RPV01}$ | Quote format for equity tranches (convert to cash via $N_{\text{tr}}^{\text{face}}$) |
| Conservation | $\sum$ tranche ETL = portfolio EL | No-arbitrage constraint |
| ETL Concavity | $\partial^2\psi/\partial K^2 \leq 0$ | Ensures positive implied loss density |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $N_{\text{port}}$ | Portfolio notional (currency) |
| $N_{\text{tr}}^{\text{face}}$ | Tranche face notional (currency), typically $N_{\text{tr}}^{\text{face}} = W \cdot N_{\text{port}}$ |
| $L(t)$ | Portfolio cumulative loss fraction at time $t$ |
| $A, D$ | Attachment, detachment points (fractions) |
| $W = D - A$ | Tranche width (fraction) |
| $\text{TL}(L)$ | Tranche loss amount (portfolio fraction units) |
| $\text{ON}(L)$ | Tranche outstanding notional (portfolio fraction units) |
| $\mathrm{ETL}(t)$ | Expected tranche loss at time $t$ |
| $\text{EON}(t)$ | Expected outstanding notional at time $t$ |
| $\psi(T,K)$ | ETL for equity tranche $[0,K]$ at horizon $T$ |
| $Q(t;A,D)$ | Tranche survival probability |
| $\mathrm{RPV01}$ | Tranche risky PV01 |
| $PV01_{\text{prem}}$ | Premium-leg PV per unit running spread (decimal per year), so $PV_{\text{prem}} = s \cdot PV01_{\text{prem}}$ |
| $Z(t)$ | Discount factor to time $t$ |
| $\alpha_i$ | Accrual fraction for period $i$ |
| $s$ | Running spread (per year) |
| $s^{\star}$ | Par spread |
| $s_0$ | Contractual spread |
| $V$ | Mark-to-market value |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the state variable for tranche pricing? | Portfolio loss $L(t)$ as a fraction of notional |
| 2 | Define the tranche loss function $\text{TL}(L)$. | $\min(\max(L-A,0), W)$ |
| 3 | What is $\text{ON}(L)$? | $W - \text{TL}(L)$, the outstanding notional |
| 4 | Define ETL. | $E[\text{TL}(L(t))]$, expected tranche loss |
| 5 | What is $\psi(T,K)$? | ETL for equity tranche $[0,K]$: $E[\min(L(T),K)]$ |
| 6 | How is any tranche's ETL related to equity ETLs? | $\mathrm{ETL}(A,D) = \psi(D) - \psi(A)$ |
| 7 | Define tranche survival $Q(t)$. | $E[1 - L_{\text{tr}}(t)] = \text{EON}(t)/W$ |
| 8 | What is the discrete protection leg PV formula? | `sum_i Z(t_i) * Delta ETL_i * N_port` |
| 9 | Why use average-notional for premium leg? | To approximate intra-period loss timing |
| 10 | What is tranche RPV01? | PV of paying $USD 1$/year on the premium leg |
| 11 | Formula for par spread? | $s^{\star} = PV_{\text{prot}} / PV01_{\text{prem}}$ |
| 12 | Formula for mark-to-market value? | $V = (s^{\star} - s_0) \times PV01_{\text{prem}}$ (for protection buyer) |
| 13 | How do equity tranches quote upfront? | Upfront (points) = $(s^{\star} - s_{\text{fixed}}) \times \mathrm{RPV01}$; upfront (cash) = points $\times N_{\text{tr}}^{\text{face}}$ |
| 14 | What is conservation of expected loss? | Sum of tranche ETLs = portfolio expected loss |
| 15 | Why are senior tranches tail-sensitive? | ETL depends on $\Pr(L \geq A)$, a tail probability |
| 16 | What does concavity of ETL in strike ensure? | Non-negative implied loss density |
| 17 | How does conditional independence help compute loss distributions? | Credits independent given factor $Z$; integrate over $Z$ |
| 18 | What's the Gaussian approximation (Shelton 2004)? | Fit conditional loss with Gaussian matching first two moments |
| 19 | Relationship between tranche and CDS pricing? | Replace issuer survival with tranche survival, use zero recovery |
| 20 | If ETL = 0 everywhere, what is par spread? | Zero |
| 21 | If ETL = $W$ at $t=0$, what happens to par spread? | Undefined (denominator → 0); upfront captures value |
| 22 | How do higher recoveries affect ETL? | Lower loss severity → lower ETL → tighter spread |
| 23 | What is the recursion formula for loss distribution? | $P_{k+1}(j) = (1-p_{k+1})P_k(j) + p_{k+1}P_k(j-\ell_{k+1})$ |
| 24 | What is the complexity of the recursion algorithm? | $O(N_c \times N_U)$ per factor value; $O(N_c^2)$ for homogeneous pools |
| 25 | In LHP model, what is the loss distribution CDF $F(K)$? | $F(K) = 1 - \Phi(A(K))$ where $A(K) = \frac{1}{\beta}(C(T) - \sqrt{1-\beta^2}\Phi^{-1}(K/(1-R)))$ |
| 26 | What is the LHP formula for equity ETL $\psi(T,K)$? | $(1-R)\Phi_2(C(T), -A(K), -\beta) + K\Phi(A(K))$ |
| 27 | Why process loss units high-to-low in the recursion? | Allows in-place array updates without storing both $P_{k-1}$ and $P_k$ |
| 28 | When should you use Monte Carlo vs analytical methods? | MC for non-standard tranches, CDO², complex structures; analytical for speed |

---

## Mini Problem Set

1. (Compute) For tranche $A=2\%$, $D=5\%$, compute $\text{TL}(L)$ and $\text{ON}(L)$ at $L=\\{0\%,2\%,4\%,5\%,8\%\\}$.
2. (Compute) Given $L(T)\in\\{1\%,4\%,9\%\\}$ with probabilities $\\{0.3,0.5,0.2\\}$, compute $\mathrm{ETL}(T;3\%,7\%)$.
3. (Concept) Show that $\text{TL}(L)=\min(L,D)-\min(L,A)$.
4. (Concept) Prove $Q(T)=\text{EON}(T)/W$.
5. (Compute) ETL(1) = 0.2%, ETL(2) = 0.5%, ETL(3) = 0.7%. Discount factors $Z=\\{0.98,0.95,0.92\\}$. Compute $PV_{\text{prot}}$ for portfolio notional USD 200mm.
6. (Desk) Explain why a senior tranche spread can jump when tail probability (of exceeding attachment) changes.
7. (Desk) Your tranche PV doesn’t match another system. List four items to check (quote format, units, curve inputs, and methodology).
8. (Concept/Compute) Show how the difference between “end-of-period notional” vs “average notional” premium PV can be expressed in terms of $\Delta\mathrm{ETL}$.
9. (Compute) A 0–3% equity tranche has par spread 1500 bp/year and $RPV01=2.5$ years. Compute upfront points for a fixed coupon of 500 bp/year.
10. (Compute/Desk) You sold tranche protection at 400 bp. Par spread is now 550 bp and $RPV01=3.8$ years. Compute the mark-to-market on USD 50mm tranche face and interpret the sign.
11. (Concept/Desk) A report shows tranche ETLs that don’t sum to portfolio expected loss. Give two possible causes.
12. (Concept) In the LHP model, explain why the “0–100%” base tranche ETL must equal portfolio expected loss (for $K\ge L_{\max}$).

### Solution Sketches (Selected)
5. $\Delta \mathrm{ETL}=\\{0.2\%,0.3\%,0.2\%\\}$. PV fraction $=0.98(0.002)+0.95(0.003)+0.92(0.002)=0.00665$. $PV_{\text{prot}}=0.00665\times USD 200\text{mm}=USD 1.33\text{mm}$.
9. Upfront points $=(0.15-0.05)\times 2.5=0.25=25\%$ of tranche notional (i.e., 25 points upfront).
10. Protection seller MTM $=(s_0-s^{\star})\times RPV01\times N_{\text{tr}} = (0.04-0.055)\times 3.8\times USD 50\text{mm}\approx -USD 2.85\text{mm}$. Negative means the position is underwater because spreads widened.
7. Example checklist: (i) bp vs decimal scaling; (ii) “points upfront” vs “price”; (iii) end-of-period vs average-notional premium; (iv) whether ETL/$Q$ is held fixed or recomputed from bumped quotes.

---

## References

- Dominic O’Kane, *Modelling Single-name and Multi-name Credit Derivatives*, “12.7 The Tranche Survival Curve”; “12.7.1 Conservation of Expected Loss”; “16.2.2 The Tranche Survival Curve”; “17.2.1 Tranche RPV01”; “20.4.1 The implied loss density”.
- John C. Hull, *Options, Futures, and Other Derivatives*, “25.10 Valuation of a Synthetic CDO”; “CDOs” (upfront quote convention).
- John C. Hull, *Risk Management and Financial Institutions*, “CDOs” (upfront quote convention; quote algebra).
- F. Neftci, *Principles of Financial Engineering*, example paragraph stating equity tranche quotes as upfront % plus 500 bp running-fee paid quarterly.
- Stéphane Crépey, *Counterparty Risk and Funding*, “10.2 CDO Tranches”.
- *Financial Risk Analytics*, “Cumulative expected tranche loss” (tranche premium/protection legs; par spread as a leg ratio).
