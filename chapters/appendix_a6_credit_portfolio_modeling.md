# Appendix A6: Credit Portfolio Modeling Beyond Base Correlation

---

## Introduction

During the 2007–2008 crisis, structured credit tranches experienced losses and spread moves that simple one-factor Gaussian copula / base correlation frameworks badly understated. The lesson for a desk quant is not “Gaussian copula is always wrong,” but rather: **base correlation is not a structural model of default dependence**. It is a quoting convention that translates tranche prices into a single “correlation” number, and it can be forced to fit prices even when the underlying dependence story is inconsistent.

The crisis period also made three limitations operationally important:
1. **Dynamics:** static copulas are one-date joint distributions; they do not explain how dependence evolves through time or through defaults.
2. **Interpolation risk:** base correlation surfaces require interpolation across attachment/detachment points, which can create arbitrage-like inconsistencies if done naively.
3. **Tail dependence:** Gaussian dependence has no tail dependence (unless correlation is exactly 1), so senior tranche risk can be systematically understated when defaults cluster.

> **Desk Reality: The 2008 Lesson**
>
> When defaults cluster, “correlation” is not the whole story. Gaussian copula dependence has **zero tail dependence** by construction, so it can dramatically underweight joint-extreme loss scenarios. Models with tail dependence (e.g., $t$-copulas, certain Archimedean copulas, or dynamic intensity models with contagion) allocate more probability mass to joint stress states, which matters most for senior tranches.
>
> The scatterplot tells the story: Gaussian copula joint defaults form a circular cloud—even with high correlation, the extreme corners (joint default/joint survival) are nearly empty. The t-copula places substantial probability mass in those corners. For a risk manager, the difference between a "50-year event" and a "7-year event" is the difference between theoretical reassurance and realized losses.

This appendix extends beyond the base correlation framework introduced in Chapter 50 to survey the broader landscape of credit portfolio models. We begin with the fundamental building blocks—portfolio loss distributions, dependence structures, and the distinction between bottom-up and top-down approaches (Section A6.1). Section A6.2 examines why base correlation "is not enough," providing a rigorous treatment of its empirical failures. Section A6.3 develops a framework map organizing the alternatives: static copula extensions, factor model enhancements, dynamic intensity models, and other mechanisms. Section A6.4 presents copula skew models in detail—the Random Factor Loading (RFL) model and the Double-t copula—that were developed specifically to address correlation smile fitting. Section A6.5 covers bespoke tranche pricing methods, a critical gap in the base correlation framework. Section A6.6 details dynamic bottom-up and top-down models. Section A6.7 addresses the Vasicek model and Basel IRB regulatory capital. Section A6.8 covers calibration and validation—the practical challenge of making these models work. Section A6.9 addresses numerical implementation and loss distribution computation methods. Section A6.10 provides step-by-step derivation pipelines with unit checks for key computational procedures. Section A6.11 covers measurement, stress testing, and sensitivity analysis. Throughout, we balance mathematical rigor with practical guidance for the desk quant who must actually implement, calibrate, and risk-manage these models.

**How to use this appendix (reading paths):**

- **If you need a “model map” first:** read A6.1–A6.3 (building blocks + why base correlation fails + taxonomy of alternatives).
- **If you’re trying to fit a correlation smile:** read A6.4 (RFL, Double-$t$) and A6.8 (calibration/validation).
- **If you’re pricing bespoke tranches:** read A6.5 and A6.9–A6.10 (loss distribution computation and implementation).
- **If you care about capital / regulatory framing:** read A6.7 (Vasicek / IRB) and A6.11 (stress/sensitivities).

**Prerequisites:** This appendix assumes familiarity with tranche mechanics (Chapter 48), expected tranche loss calculations (Chapter 49), and the Gaussian copula framework (Chapter 50). Readers should be comfortable with measure-theoretic probability at the level of Appendix A1.

---

## Conventions and Notation

| Symbol | Definition |
|--------|------------|
| $N_C$ | Number of credits in reference portfolio |
| $\tau_i$ | Default time of credit $i$ |
| $Q_i(T) = \mathbb{P}(\tau_i \gt T)$ | Survival probability to time $T$ |
| $p_i(T) = 1 - Q_i(T)$ | Default probability to time $T$ |
| $\lambda_i(t)$ | Hazard rate (instantaneous default intensity) |
| $L(T)$ | Fractional portfolio loss at time $T$ |
| $L_{i}$ | Loss-given-default for credit $i$ (fraction of notional) |
| $R_i$ | Recovery rate; $L_i = 1 - R_i$ |
| $K, K_1, K_2$ | Tranche attachment/detachment points |
| $\psi(T, K) = \mathbb{E}[\min(L(T), K)]$ | Expected tranche loss (base tranche to strike $K$) |
| $\rho$ | Correlation parameter in one-factor models |
| $X$ | Systematic (market) factor |
| $\varepsilon_i$ | Idiosyncratic factor for credit $i$ |
| $Z$ | Latent variable in threshold models |
| $\Phi(\cdot), \Phi^{-1}(\cdot)$ | Standard normal CDF and inverse |
| $\phi(\cdot)$ | Standard normal density |
| $\Phi_2(a, b; \rho)$ | Bivariate normal CDF with correlation $\rho$ |
| TLP | Tranche Loss Proportion |
| RFL | Random Factor Loading |
| ETL | Expected Tranche Loss |
| WCDR | Worst Case Default Rate |

---

## A6.1 Core Concepts: The Building Blocks

### A6.1.1 Portfolio Loss Distribution

The portfolio loss $L(T)$ at horizon $T$ is the central object of interest. For a portfolio of $N_C$ credits with exposures $w_i$ (summing to 1) and loss-given-defaults $L_i$:

$$\boxed{L(T) = \sum_{i=1}^{N_C} w_i L_i \mathbf{1}_{\\{\tau_i \leq T\\}}}$$

The **loss distribution** is the probability law of $L(T)$. McNeil, Frey, and Embrechts in *Quantitative Risk Management* emphasize that "the main quantity of interest in portfolio credit risk is the distribution of portfolio losses over a fixed time horizon."

For tranche pricing, we need the expected loss on tranches of the form $[K_1, K_2]$:

$$\mathbb{E}[L(T; K_1, K_2)] = \frac{\mathbb{E}[\min(L(T), K_2)] - \mathbb{E}[\min(L(T), K_1)]}{K_2 - K_1}$$

This depends on the *entire distribution* of $L(T)$, not just its mean.

**Key insight:** Two portfolios with identical expected losses but different loss *distributions* will produce different tranche prices. The challenge is to model the distribution, not just the mean.

> **Why Copulas? An Intuition Box**
>
> If you understand single-name CDS pricing from Chapter 41, you know how to model one firm's default. But a portfolio has 125 names. How do they default together? A copula is the mathematical glue that couples individual default behaviors into a joint distribution. It answers: given that firm A defaulted, how does that change the probability that firm B defaults?
>
> Think of the economy as having good days and bad days. On bad days (low factor realization), more firms are near their default thresholds. A factor model says: conditional on knowing whether it's a good or bad day, each firm's fate is independent. The common factor is what makes defaults cluster.

### A6.1.2 Dependence Modeling: Why Correlation Is Not Enough

The term "correlation" is used loosely in credit markets, but McNeil et al. clarify the distinction between different dependence concepts:

**Default correlation:** The correlation between default indicators:
$$\rho_{ij}^{def}(T) = \mathrm{Corr}(I(\tau_i \leq T), I(\tau_j \leq T))$$

**Asset correlation:** In structural models, the correlation between latent asset values:
$$\rho_{ij}^{\text{asset}} = \text{Corr}(Z_i, Z_j)$$

**Copula correlation:** The correlation parameter in a copula specification.

These are *not* the same. As McNeil et al. note, "linear correlation is not a particularly useful measure of dependence for non-elliptical distributions." Credit applications typically involve highly non-normal (indeed, binary) outcomes where linear correlation captures only a fraction of the dependence structure.

**Tail dependence** is often more relevant for senior tranches. McNeil et al. define the lower tail dependence coefficient as:

$$\lambda_L = \lim_{u \to 0^+} \mathbb{P}(U_2 \leq u \mid U_1 \leq u)$$

where $(U_1, U_2)$ have copula $C$. This measures the probability of joint extreme events in the lower tail—precisely what matters for credit portfolio losses.

**The Gaussian copula's fatal flaw:** McNeil et al. prove that $\lambda_L = 0$ for the Gaussian copula whenever $\rho \lt 1$. No matter how high the correlation, as we move into the extreme tail, joint defaults appear to occur independently. This is a critical limitation for senior tranches, which are exposed precisely to these tail events.

**The t-copula alternative:** The Student's t-copula has non-zero tail dependence. McNeil et al. (Example 5.33) derive the closed-form expression:

$$\boxed{\lambda = 2t_{\nu+1}\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)}$$

where $t_{\nu+1}$ is the CDF of the Student's t distribution with $\nu+1$ degrees of freedom.

**Table A6.1: Tail dependence coefficients for the t-copula** (from McNeil et al. Table 5.1)

| $\nu$ | $\rho = -0.5$ | $\rho = 0$ | $\rho = 0.5$ | $\rho = 0.9$ |
|-------|---------------|------------|--------------|--------------|
| 2 | 0.06 | 0.18 | 0.39 | 0.72 |
| 4 | 0.01 | 0.08 | 0.25 | 0.63 |
| 10 | 0.00 | 0.01 | 0.08 | 0.46 |
| $\infty$ (Gaussian) | 0 | 0 | 0 | 0 |

The table reveals several important features. First, even with zero or negative correlation, the t-copula exhibits some tail dependence—this reflects the fact that extreme events in one margin make extreme events in the other more likely through the shared heavy-tailed factor. Second, tail dependence increases as degrees of freedom decrease (fatter tails). Third, the Gaussian copula ($\nu = \infty$) has zero tail dependence regardless of correlation.

**Practical implication:** McNeil et al. compute that for a meta-t distribution with $\nu = 4$ and $\rho = 0.5$, the probability of both variables exceeding their 99.5% quantiles is 2.79 times higher than under the Gaussian copula. For a risk manager, "50-year events and 7-year events have a very different significance."

### A6.1.3 Copula Models for Default Dependence

Copulas separate marginal distributions from dependence structure. For default times, this separation is particularly valuable because:

1. **Marginal default probabilities** are well-calibrated from single-name CDS
2. **Joint default behavior** is the modeling challenge

**Definition (Copula):** A copula $C: [0,1]^{N_C} \to [0,1]$ is a multivariate distribution function with uniform marginals. By Sklar's theorem, the joint distribution of $(\tau_1, \ldots, \tau_{N_C})$ can be written:

$$\mathbb{P}(\tau_1 \gt t_1, \ldots, \tau_{N_C} \gt t_{N_C}) = C(Q_1(t_1), \ldots, Q_{N_C}(t_{N_C}))$$

**Common copulas in credit:**

| Copula | Parameters | Tail Dependence | Use Case |
|--------|------------|-----------------|----------|
| Gaussian | Correlation matrix $\Sigma$ | None ($\lambda_L = \lambda_U = 0$) | Market standard |
| Student's $t$ | $\Sigma$, degrees of freedom $\nu$ | Symmetric: $\lambda_L = \lambda_U \gt 0$ | Fat-tail modeling |
| Clayton | $\theta \gt 0$ | Lower only: $\lambda_L = 2^{-1/\theta}$ | Asymmetric downside |
| Gumbel | $\theta \gt 1$ | Upper only: $\lambda_U = 2 - 2^{1/\theta}$ | Asymmetric upside |
| Archimedean | Generator $\psi$ | Varies | Flexible structures |

**Archimedean copula structure:** McNeil et al. (QRM Ch. 5) define the Archimedean copula through a generator function $\psi$:

$$C(u_1, \ldots, u_n) = \psi^{-1}\\!\left(\psi(u_1) + \cdots + \psi(u_n)\right)$$

where $\psi: [0,1] \to [0,\infty]$ is a continuous, strictly decreasing function with $\psi(1) = 0$. Different choices of $\psi$ yield different copula families:
- **Clayton:** $\psi(u) = u^{-\theta} - 1$ for $\theta \gt 0$ → lower tail dependence
- **Gumbel:** $\psi(u) = (-\ln u)^\theta$ for $\theta \geq 1$ → upper tail dependence

QRM illustrates qualitative tail behavior differences across copulas: Gumbel produces upper tail dependence, Clayton produces lower tail dependence, and Gaussian produces none.

**Clayton copula tail dependence:** McNeil et al. (QRM, Example 5.31) derive the lower tail dependence coefficient for the Clayton copula:

$$\boxed{\lambda_L^{\text{Clayton}} = 2^{-1/\theta} \quad \text{for } \theta \gt 0}$$

For $\theta = 2$, this gives $\lambda_L = 2^{-0.5} \approx 0.71$—substantial tail dependence even with moderate parameter values. The Clayton copula has no upper tail dependence ($\lambda_U = 0$), making it appropriate for modeling situations where joint extreme losses (lower tail) are more likely than joint extreme gains. This asymmetry can be valuable for credit portfolios where crisis contagion is a primary concern.

### A6.1.4 Factor Models

Factor models reduce the dimensionality of dependence by assuming that default correlation arises through exposure to common factors. The canonical one-factor model (Chapter 50) has:

$$Z_i = \sqrt{\rho} X + \sqrt{1-\rho} \varepsilon_i$$

where $X$ is the systematic factor and $\varepsilon_i$ are idiosyncratic shocks.

**Multi-factor extensions** allow for richer correlation structures:

$$Z_i = \sum_{k=1}^{K} \beta_{ik} X_k + \sqrt{1 - \sum_{k=1}^{K} \beta_{ik}^2} \varepsilon_i$$

where $X_k$ might represent industry, region, or rating factors. For a two-factor model, the correlation between credits $i$ and $j$ is:

$$\boxed{c_{ij} = \beta_{1i}\beta_{1j} + \beta_{2i}\beta_{2j}}$$

**Sector correlation example (O'Kane Ch. 13):** Consider four credits grouped into two sectors (A and B, two credits each). With a two-factor model where:
- All credits have equal exposure to the first (market) factor: $\beta_{1a} = \beta_{1b} = 0.6$
- Sector A credits are negatively correlated to the second factor: $\beta_{2a} = -0.4$
- Sector B credits are positively correlated to the second factor: $\beta_{2b} = 0.5$

The resulting correlation matrix is:

$$C = \begin{pmatrix}
1.00 & 0.52 & 0.16 & 0.16 \\
0.52 & 1.00 & 0.16 & 0.16 \\
0.16 & 0.16 & 1.00 & 0.61 \\
0.16 & 0.16 & 0.61 & 1.00
\end{pmatrix}$$

O'Kane notes: "We see that there is a strong positive correlation within each of sectors A and B and that there is a lower positive correlation between sectors. This is exactly the sort of correlation structure that we require if we are to capture the effect of sectors."

**Key insight:** O'Kane emphasizes that modeling $M$ sectors requires an $M$-factor model. A one-factor model cannot capture sector-specific correlation patterns. However, "the price of most portfolio credit derivatives depends to first order on the average correlation level rather than the correlation factor structure," which partially justifies the one-factor approximation in practice.

**Conditional independence:** Crucially, conditional on the factors $(X_1, \ldots, X_K)$, defaults are independent. This enables tractable computation via:

$$\mathbb{E}[g(L)] = \mathbb{E}_X[\mathbb{E}[g(L) \mid X]]$$

where the inner expectation involves independent Bernoulli trials.

### A6.1.5 Bottom-Up vs. Top-Down Approaches

Credit portfolio models divide into two fundamental philosophies:

**Bottom-up models** start from individual credits:
- Model default times $(\tau_1, \ldots, \tau_{N_C})$ directly
- Aggregate to portfolio loss: $L(T) = \sum_i w_i L_i \mathbf{1}_{\\{\tau_i \leq T\\}}$
- Examples: Gaussian copula, structural models, intensity-based models

**Advantages:**
- Natural calibration to single-name CDS
- Straightforward interpretation
- Hedging with single-name instruments

**Top-down models** start from the portfolio loss process:
- Model $L(t)$ or the number of defaults $N(t)$ directly
- No explicit modeling of individual defaults
- Examples: Markov chain models, loss process intensity models

**Advantages:**
- Natural fit to tranche prices
- Can capture complex portfolio dynamics
- Often more tractable for exotic products

O'Kane notes that "the choice between bottom-up and top-down approaches depends on the products being priced and the hedging strategy."

### A6.1.6 Static vs. Dynamic Models

**Static models** (like the Gaussian copula) specify the joint distribution of default times at a single horizon. They answer: "What is the probability of $k$ defaults by time $T$?"

**Dynamic models** specify the evolution of default intensities or the loss process through time. They answer: "How does the probability of future defaults change as information arrives?"

Dynamic models are essential for:
- Forward-starting tranches
- Tranche options
- Mark-to-market risk management
- Understanding spread dynamics after defaults

As O'Kane emphasizes, static models "do not tell us how the correlation changes through time or how defaults cause the spreads of surviving credits to widen."

### A6.1.7 Model Selection Decision Tree

Choosing among the many available models requires understanding their trade-offs. The following decision tree synthesizes guidance from O'Kane's model comparison:

```
Need multi-maturity pricing (forward tranches, options)?
├── YES → Dynamic model required
│   ├── Want name-level structure? → IG or AJD (Section A6.6)
│   │   ├── Simple jump clustering → Intensity Gamma
│   │   └── Correlated diffusive + jump intensities → AJD
│   └── Portfolio-first approach? → Markov chain (Section A6.6)
└── NO → Static copula may suffice
    ├── Need to fit skew? → RFL, Double-t, or other skew model (Section A6.4)
    │   ├── State-dependent correlation → RFL
    │   └── Fat-tailed factor structure → Double-t
    └── Single tranche/simple exposure? → Base correlation (with warnings)
```

**Table A6.2: Model Comparison Summary**

| Model | Tail Dependence | Fits Skew? | Multi-Maturity? | Bespoke? | Complexity |
|-------|-----------------|------------|-----------------|----------|------------|
| Gaussian | No | No | Limited | Yes | Low |
| Student-t | Yes | Partial | Limited | Yes | Low |
| RFL | Adjustable | Yes | No | Yes | Medium |
| Double-t | Yes | Yes | No | Yes | Medium |
| IG | Via jumps | Yes | Yes | Limited | High |
| Markov chain | Embedded | Yes | Yes | No | High |

---

## A6.2 Why Base Correlation Is Not Enough

### A6.2.1 The Correlation Smile and Its Interpretation

Recall from Chapter 50 that base correlation typically increases with strike:

| Strike | Base Correlation (typical) |
|--------|---------------------------|
| 3% | 20-25% |
| 7% | 30-40% |
| 10% | 40-50% |
| 15% | 50-60% |
| 30% | 70-80%+ |

This upward-sloping "smile" reveals that the Gaussian copula cannot simultaneously fit tranches across the capital structure with a single correlation parameter.

**Interpretation 1: Missing tail dependence.** The Gaussian copula's zero tail dependence forces senior tranches to require implausibly high correlations to generate sufficient expected loss. The smile is the market's way of compensating for the model's limitation.

**Interpretation 2: Wrong distributional assumptions.** The model may be misspecified—the true loss distribution has fatter tails or different skewness than Gaussian copula permits.

**Interpretation 3: Model risk premium.** Market participants may be charging a premium for model uncertainty, particularly for senior tranches where model risk is largest.

> **Desk Reality: What the Correlation Smile Tells Traders**
>
> When traders see a steep correlation smile (base correlation rising sharply with strike), they know senior tranches are expensive relative to equity. This can signal:
> - Fear of systemic events (buying protection on seniors)
> - Dealers hedging CDO warehouses (selling equity, buying senior)
> - Regulatory capital arbitrage unwinding
>
> Conversely, a flat smile suggests the market sees idiosyncratic rather than systemic risk. Correlation desks trade the shape of the smile, not just its level.

### A6.2.2 Arbitrage and Interpolation Pathologies

O'Kane identifies a fundamental problem: "base correlation is not a consistent pricing framework."

**The interpolation problem:** Base correlations are only observable at traded strikes (e.g., 3%, 7%, 10%, 15%, 30%). To price bespoke tranches with different strikes, one must interpolate. But:

1. **Linear interpolation in base correlation space** can violate no-arbitrage constraints
2. **No-arbitrage interpolation** requires constraints on the expected tranche loss function $\psi(T, K)$

The no-arbitrage conditions from O'Kane (Chapter 19) require:

$$\frac{\partial \psi(T, K)}{\partial K} \in [0, 1]$$
$$\frac{\partial^2 \psi(T, K)}{\partial K^2} \leq 0$$

Interpolating base correlations does not guarantee these conditions hold.

**Worked Example A6.1: Arbitrage from Naive Interpolation**

Suppose the 3% and 7% base tranches have base correlations of 25% and 35% respectively. Linear interpolation gives $\rho^{\text{base}}(5\\%) = 30\\%$.

Under the Gaussian copula with realistic parameters (5-year maturity, 50bp average spread, 40% recovery):
- $\psi(5Y, 3\\%)$ implied by $\rho = 25\\%$: approximately 1.2%
- $\psi(5Y, 5\\%)$ implied by $\rho = 30\\%$: approximately 1.6%
- $\psi(5Y, 7\\%)$ implied by $\rho = 35\\%$: approximately 2.3%

Check convexity: The second differences should be non-positive.
$(1.6 - 1.2)/2 = 0.20$ (slope 3% to 5%)
$(2.3 - 1.6)/2 = 0.35$ (slope 5% to 7%)

The slope is *increasing*—violating convexity and permitting arbitrage through a butterfly trade.

### A6.2.3 ETL Interpolation: A Better Approach

O'Kane develops an alternative interpolation strategy that works in **expected tranche loss (ETL) space** where no-arbitrage constraints are more easily enforced:

**The ETL interpolation process:**
1. Calibrate base correlations $\rho(K)$ at standard strikes (e.g., 3%, 7%, 10%, 15%, 30%)
2. For each strike, compute $\psi(T, K) = \mathbb{E}_{\rho(K)}[\min(L(T), K)]$
3. Add boundary conditions: $\psi(T, 0) = 0$ and $\psi(T, L_{\max}) = \mathbb{E}[L(T)]$
4. Interpolate in ETL space using a monotonic scheme (e.g., PCHIP spline)
5. Invert to recover implied base correlations at all strikes

**Why ETL interpolation works better:** The no-arbitrage conditions $\partial\psi/\partial K \in [0,1]$ and $\partial^2\psi/\partial K^2 \leq 0$ are linear constraints in ETL space. O'Kane notes: "the ETL interpolation is more arbitrage free than the direct interpolation of the base correlation curve using the linear and cubic splines."

**Limitations:** O'Kane cautions that even ETL interpolation does not guarantee arbitrage-free pricing. For CDX, the ETL skeleton itself may violate concavity at high strikes, leading to residual arbitrage that "has not been caused by the interpolation" but by the model or recovery assumptions.

**Practical guidance:** O'Kane recommends using the Piecewise Cubic Hermite Interpolant (PCHIP) spline, which guarantees smoothness and monotonicity (though not concavity). The resulting tranchelet spreads are "a smooth function of their subordination" with "no negative tranchelet spreads."

### A6.2.4 Lack of Spread Dynamics

The Gaussian copula is a static model: it specifies the joint distribution at maturity but says nothing about how spreads evolve through time.

**The "spread dispersion" problem:** After a default, spreads on surviving credits should widen—both because of correlation (shared factor exposure) and because of contagion effects. The static model provides no framework for this.

O'Kane notes that within the latent variable framework, "the only events that we can observe are the defaults of credits. This does not mean that there are no spread dynamics at all... but they are implicit rather than explicit."

**Implication for hedging:** Delta hedges computed from the static model may underperform because they don't account for how the correlation surface itself moves.

### A6.2.5 Default-Induced Spread Dynamics in the Latent Variable Model

Although the latent variable model is static, it implies *implicit* spread dynamics when we condition on observed defaults. O'Kane derives the conditional survival curve for credit B given that credit A defaults at time $t$:

$$\boxed{\hat{Q}_B(t, T) = 1 - \frac{\Phi\left(\frac{C_B(T) - \rho C_A(t)}{\sqrt{1-\rho^2}}\right) - \Phi\left(\frac{C_B(t) - \rho C_A(t)}{\sqrt{1-\rho^2}}\right)}{1 - \Phi\left(\frac{C_B(t) - \rho C_A(t)}{\sqrt{1-\rho^2}}\right)}}$$

where $C_i(T) = \Phi^{-1}(1 - Q_i(T))$ is the default threshold.

**Key observations from O'Kane:**

1. **Zero correlation:** When $\rho = 0$, we have $\hat{Q}_B(t,T) = Q_B(t,T)$—the default of A has no impact on B's survival curve. "This makes sense: if there is no correlation between A and B, the spread curve of credit B should not change after a default of credit A."

2. **Positive correlation:** The conditional spread curve jumps *up* when A defaults. The magnitude increases with $\rho$.

3. **Negative correlation:** The conditional spread curve jumps *down*—B becomes safer when A defaults.

4. **Time decay:** "The size of the spread jump declines with $t = \tau_A$, i.e., how far in the future credit A defaults." This reveals non-stationary behavior in the Gaussian model.

**Implication for hedging:** This implicit spread dynamic affects how the portfolio responds after defaults. A proper dynamic model would make these effects explicit and hedgeable.

### A6.2.6 The Base Correlation Surface

When 3Y, 5Y, 7Y, and 10Y tranche quotes are available, O'Kane proposes building a **base correlation surface** $\rho(T, K)$:

**Bootstrap approach:**
1. Use 3Y quotes to calibrate $\rho(3, K)$ at standard strikes, assuming piecewise flat structure from $T=0$ to $T=3$
2. For 5Y tranches, solve for flat base correlation between $T=3$ and $T=5$ that reprices the market, using $\rho(3, K)$ for cash flows before $T=3$
3. Repeat for 7Y and 10Y maturities
4. Apply ETL interpolation in strike dimension at each maturity

**No-arbitrage in the time dimension:** The constraint is:
$$\frac{\partial^2 \psi(T, K)}{\partial T \partial K} \geq 0$$

This cross-derivative constraint is difficult to enforce directly. O'Kane suggests enforcing the weaker condition $\partial\psi/\partial T \geq 0$ (ETL increases with maturity), allowing a two-stage construction: strike interpolation first, then maturity interpolation.

### A6.2.7 Crisis Behavior and Regime Dependence

During the 2007-2008 crisis:
- Base correlations for equity tranches spiked (defaults became "more correlated")
- Base correlations for senior tranches also increased dramatically
- The entire correlation surface shifted in ways the static model couldn't predict

McNeil et al. observe that "in periods of market stress, dependence between assets tends to increase," but the Gaussian copula has no mechanism for time-varying or state-dependent correlation.

---

## A6.3 Framework Map: Beyond Base Correlation

Following O'Kane's taxonomy, we organize alternative approaches into four categories:

### A6.3.1 Category A: Static Copula Extensions

**A1. Alternative marginal distributions**
- Replace Gaussian marginals with Student's t or other fat-tailed distributions
- Preserves tractability of one-factor structure
- Addresses some tail concerns but not fundamental copula limitations

**A2. Alternative copulas**
- Student's t copula with tail dependence
- Clayton copula for asymmetric lower-tail dependence
- Archimedean copulas for flexible structures

**A3. Random factor loading (RFL)** (Section A6.4.1)
- Allow the factor loading $\rho$ to be stochastic
- Generates additional correlation randomness
- Can match more tranche prices but adds parameters

**A4. Stochastic recovery**
- Recovery rates correlated with the systematic factor
- "In a bad economy, both defaults increase and recovery rates fall"
- Increases senior tranche risk without changing correlation

**A5. Multi-factor models**
- Industry or regional factors
- Richer correlation structure
- More realistic but harder to calibrate

**A6. Marshall-Olkin (common-shock) copula**
- $M$ independent Poisson shock processes with intensities $\lambda_m$; shock-name incidence matrix $I_{im} \in \\{0,1\\}$
- Common shocks: one jump can trigger defaults of multiple names (subset-specific)
- Captures simultaneous default / jump-to-default clustering with an interpretable mechanism
- **Limitation:** Parameter explosion; calibration identification can be difficult
- Calibrate shock intensities and incidence structure to tranche prices and/or joint-default statistics

### A6.3.2 Category B: Factor Model Enhancements

**B1. Extended one-factor models**
- Allow time-varying or state-dependent correlation
- Example: correlation increases when factor is low

**B2. Threshold models and Bernoulli mixture representations (LT-Archimedean)**
- Mixing variable $\Psi$ (systematic), conditional Bernoulli defaults given $\Psi$
- Dependence induced by randomness of $\Psi$; conditional independence given $\Psi$
- QRM explicitly links LT-Archimedean copulas to threshold/mixture models and shows independence and comonotonic limits for the Clayton parameter ($\theta \to 0$ → independence; $\theta \to \infty$ → comonotonicity)
- Use: Portfolio risk (VaR/ES), stress testing, exploring tail sensitivity to mixing distribution choice

**B3. Levy factor models**
- Replace Gaussian factor with Levy process
- Generates jumps in default clustering
- More analytically tractable than full dynamic models

**B4. Double-t copula** (Section A6.4.2)
- Both factor and idiosyncratic shocks are t-distributed
- Greater flexibility in tail behavior

**B5. Random recovery factor models**
- Recovery rate driven by systematic factor
- $R_i \mid X = f(X)$ where $f$ is decreasing
- Captures "bad states have low recovery" intuition

### A6.3.3 Category C: Dynamic Intensity Models

**C1. Intensity Gamma model** (Joshi and Stacey) (Section A6.6.1)
- Time-changed default process with jumps
- Generates default clustering
- Relatively tractable calibration

**C2. Affine Jump Diffusion (AJD)** (Duffie and Garleanu) (Section A6.6.2)
- Default intensities follow correlated jump-diffusions
- Common jumps create clustering
- Semi-analytical calibration possible

**C3. Contagion models** (Section A6.6.3)
- Defaults increase intensities of surviving credits
- "Infectious default" mechanisms
- Captures crisis propagation

### A6.3.4 Category D: Top-Down and Other Mechanisms

**D1. Markov chain models** (Schönbucher) (Section A6.6.4)
- Model portfolio loss as a Markov process
- Direct calibration to tranche prices
- Natural for forward-starting products

**D2. Loss process intensity models**
- Model the intensity of the portfolio loss process directly
- Skip individual-name modeling
- Efficient for portfolio-level products

**D3. Structural extensions**
- Return to Merton-style models with richer dynamics
- Correlated asset processes
- First-passage time defaults

**D4. Hybrid structural-reduced-form models**
- O'Kane describes hybrid models inspired by structural "firm value" dynamics but used as reduced-form models calibrated to market prices (no capital-structure interpretation)
- Correlate firm-value-like state variables to induce default dependence; use rich dynamics to fit skew

> **Implementation note (model-choice dependent):** Hybrid structural–reduced-form models come in many variants. To write a concrete pricing/calibration recipe (not just a schematic description), we need the exact model equations and calibration targets you want this book to follow (desk model spec or a specific reference section).

**D5. Credit migration Markov chains (ratings transitions)**
- Hull discusses rating transition matrices and notes that independence across periods is an approximation; "ratings momentum" can violate strict independence
- Use: Risk models (CreditMetrics-style) that incorporate downgrades and default as multi-state Markov processes

> **Scope note:** This appendix focuses on default dependence. If you need *correlated migrations* (multi-name rating dynamics), you need an explicit multi-name transition model (or a correlation structure on latent drivers) beyond the one-name transition-matrix setup.

### A6.3.5 Industry Models: CreditRisk+

While the categories above focus on copula and intensity-based approaches, the industry model **CreditRisk+** (Credit Suisse Financial Products, 1997) takes a fundamentally different approach based on Poisson mixture distributions.

**Structure of CreditRisk+:** McNeil et al. (QRM Section 8.4.2) describe CreditRisk+ as "a Poisson mixture model where the factor vector $\boldsymbol{\Psi}$ consists of $p$ independent, gamma-distributed random variables." The key features are:

1. **Conditional Poisson defaults:** Credit $i$ has a stochastic default intensity $\lambda_i(\boldsymbol{\Psi}) = k_i \boldsymbol{w}_i' \boldsymbol{\Psi}$, where:
   - $k_i \approx$ default probability (for small $k_i$)
   - $\boldsymbol{w}_i$ = vector of factor weights (summing to 1)
   - $\boldsymbol{\Psi}$ = independent gamma-distributed factors with $\text{Ga}(\sigma_j^{-2}, \sigma_j^{-2})$

2. **Gamma-Poisson mixture:** Conditional on $\boldsymbol{\Psi}$, defaults are independent Poisson. The marginal distribution follows from the gamma mixture:

$$\tilde{M} \mid \boldsymbol{\Psi} = \boldsymbol{\psi} \sim \text{Poi}\left(\sum_{i=1}^{m} k_i \boldsymbol{w}_i' \boldsymbol{\psi}\right)$$

McNeil et al. show that for $p$ gamma factors, "`Mtilde` is equal in distribution to a sum of $p$ independent negative binomial random variables."

3. **Panjer recursion:** The loss distribution can be computed efficiently using actuarial techniques. McNeil et al. note: "Using Panjer recursion, it is possible to derive simple recursion formulas for the probabilities `P(Mtilde = k)`."

**Advantages:**
- Closed-form loss distribution (no Monte Carlo needed)
- Natural connection to actuarial portfolio theory
- Tractable for large portfolios

**Limitations:**
- No explicit copula structure
- Limited flexibility in tail behavior
- Less intuitive than factor models for credit practitioners

**Connection to copula models:** McNeil et al. establish that "if the $V_i$ follow a gamma distribution with mean one, the static version of a generalized LT-Archimedean factor copula model is the popular CreditRisk+ model." This provides a bridge between the Poisson mixture and copula frameworks.

---

## A6.4 Copula Skew Models: RFL and Double-t

The standard one-factor Gaussian model cannot fit the correlation skew because it has only one parameter. This section presents two models developed specifically to address this limitation.

### A6.4.1 The Random Factor Loading (RFL) Model

**Motivation:** O'Kane (Ch. 21.10) introduces the Random Factor Loading model of Andersen and Sidenius (2004): "The RFL model by Andersen and Sidenius (2004) is a one-factor latent variable model in which there is a weighting on the systemic factor which is a deterministic function of the random market factor $Z$."

**Model specification:** The latent variable for credit $i$ is:

$$\boxed{A_i = a_i(Z)Z + b_i\varepsilon_i + m_i}$$

where:
- $Z$ and $\varepsilon_i$ are independent standard normal random variables
- $a_i(Z)$ is the **factor loading function**—deterministic but state-dependent
- $b_i$ and $m_i$ enforce $\mathbb{E}[A_i] = 0$ and $\text{Var}(A_i) = 1$

The key constraints are:
$$\mathbb{E}[A_i] = 0 \Rightarrow m_i = -\mathbb{E}[a_i(Z)Z]$$
$$\text{Var}(A_i) = 1 \Rightarrow \mathbb{E}[a_i(Z)^2 Z^2] + b_i^2 = 1$$

**The single-step factor loading function:** O'Kane presents a tractable specification where the factor loading takes two values:

$$a_i(Z) = \begin{cases} \alpha_i & \text{if } Z \lt \Theta_i \\ \beta_i & \text{if } Z \geq \Theta_i \end{cases}$$

This creates state-dependent correlation:
- **If $\alpha_i \gt \beta_i$ and $\Theta_i \lt 0$**: Higher correlation in bad states (low $Z$), lower correlation in good states
- **If $\alpha_i \lt \beta_i$ and $\Theta_i \lt 0$**: Lower correlation in bad states, higher in good states

**Why this fits the skew:** O'Kane explains: "By weighting $Z$ by a factor weight $a(Z)$, the random factor loading model has a correlation which is a function of $Z$."

- If $a_i(Z) \gt 0$ and *increasing* in $Z$: correlation increases in good states, decreases in bad states
- If $a_i(Z) \gt 0$ and *decreasing* in $Z$: correlation decreases in good states, increases in bad states

The second case—higher correlation in crisis states—fits the empirical observation that defaults cluster more during downturns.

**Calibration results (O'Kane Table 21.4):** Calibrating to CDX NA IG Series 7 tranches:

| Parameter | Calibrated Value |
|-----------|------------------|
| $\alpha$ | 54.25% |
| $\beta$ | 31.16% |
| $\Theta$ | -2.58 |

**Interpretation:**
- $\alpha \gt \beta$: Higher correlation in bad states (when $Z \lt -2.58$)
- The threshold $\Theta = -2.58$ corresponds to the 0.49th percentile—the model switches to high-correlation mode in extreme downturns

**Tranche pricing fit:**

| Tranche | Market Upfront | Market Spread | RFL Upfront | RFL Spread |
|---------|----------------|---------------|-------------|------------|
| 0-3% | 24.88 | 500 | 24.88 | 500 |
| 3-7% | — | 90 | — | 123 |
| 7-10% | — | 19 | — | 20 |
| 10-15% | — | 8 | — | 5 |
| 15-30% | — | 3 | — | 1 |

The fit is reasonable but not perfect—O'Kane notes that "the quality of the market fit improves significantly when an additional step is added to the factor loading function $a_i(Z)$."

> **Desk Reality: Why RFL Matters for Correlation Trading**
>
> The RFL model captures something the Gaussian copula misses: correlation regimes. In normal markets, defaults appear relatively independent. In crisis markets, everything correlates. A correlation trader using RFL can see that:
> - Equity tranches are short this regime effect (they lose more in high-correlation regimes)
> - Senior tranches are long the regime effect (they lose more when the model switches to high-correlation mode)
>
> This insight helps explain why senior tranches can gap down suddenly—the market may be repricing the probability of entering the high-correlation regime.

### A6.4.2 The Double-t Copula Model

**Motivation:** The standard t-copula replaces Gaussian factors with t-distributed factors, generating tail dependence. The Double-t copula extends this by making *both* the systematic and idiosyncratic factors t-distributed.

**Model specification (O'Kane Ch. 21.6, Hull and White 2003b):**

$$\boxed{A_i = \beta_i \sqrt{\frac{\nu_Z - 2}{\nu_Z}} Z + \sqrt{1 - \beta_i^2} \sqrt{\frac{\nu_\varepsilon - 2}{\nu_\varepsilon}} \varepsilon_i}$$

where:
- $Z \sim t_{\nu_Z}$ (t-distributed systematic factor with $\nu_Z$ degrees of freedom)
- $\varepsilon_i \sim t_{\nu_\varepsilon}$ (t-distributed idiosyncratic factor with $\nu_\varepsilon$ degrees of freedom)
- $Z$ and $\varepsilon_i$ are independent
- The normalization factors ensure $\text{Var}(A_i) = 1$

**Key feature:** This is *not* a t-copula because the sum of two t-distributed random variables is not t-distributed. O'Kane notes: "It is therefore known as the Double-t copula model."

**Why Double-t fits the skew:** The model has three free parameters ($\rho$, $\nu_Z$, $\nu_\varepsilon$) versus one for Gaussian. O'Kane explains the role of each:

1. **$\nu_Z$ (systematic factor df):** Controls the probability of extreme market-wide shocks
2. **$\nu_\varepsilon$ (idiosyncratic factor df):** Controls the probability of extreme name-specific shocks
3. **$\rho$ (correlation):** Controls the overall dependence level

**Calibration results (O'Kane Table 21.1):** Calibrating to CDX NA IG:

| Parameter | Calibrated Value |
|-----------|------------------|
| $\nu_Z$ | 7.0 |
| $\nu_\varepsilon$ | 2.5 |
| $\rho$ | 15.0% |

| Tranche | Market Upfront | Market Spread | Double-t Upfront | Double-t Spread |
|---------|----------------|---------------|------------------|-----------------|
| 0-3% | 24.88 | 500 | 24.99 | 500 |
| 3-7% | — | 90 | — | 98 |
| 7-10% | — | 19 | — | 16 |
| 10-15% | — | 8 | — | 6 |
| 15-30% | — | 3 | — | 3 |

**Interpreting the calibration:**
- $\nu_\varepsilon = 2.5$ (very fat-tailed): Individual credits can experience extreme jumps
- $\nu_Z = 7.0$ (moderately fat-tailed): Market-wide shocks are less extreme but still fatter than Gaussian
- $\rho = 15\\%$ (low correlation): Much lower than typical Gaussian copula correlations because the fat tails already generate dependence

**Comparison with RFL (O'Kane):** The Double-t tranchelet spreads "do not flatten out but instead fall slowly to zero at the maximum loss. This reflects the very long tailed shape of the Double-t copula loss distribution." In contrast, RFL spreads fall to zero more quickly.

### A6.4.3 Model Comparison: Copula Skew Models

O'Kane (Ch. 21.12) compares tranche leverage ratios across models:

| Tranche | Mixing | CBM | RFL | Double-t | Implied | Base Corr |
|---------|--------|-----|-----|----------|---------|-----------|
| 0-3% | 26.97 | 25.16 | 24.40 | 22.83 | 30.37 | 23.03 |
| 3-7% | 5.87 | 7.73 | 7.73 | 6.67 | 4.26 | 5.45 |
| 7-10% | 1.17 | 1.10 | 0.94 | 1.76 | 0.29 | 1.38 |
| 10-15% | 0.34 | 0.04 | 0.02 | 0.50 | 0.01 | 0.23 |
| 15-30% | 0.09 | 0.01 | 0.00 | 0.08 | 0.00 | 0.04 |

**Key observations:**
- Double-t assigns more risk to mezzanine and senior tranches than RFL
- The implied copula model concentrates risk in equity
- Leverage ratios vary significantly across models—this is model risk

### A6.4.4 Copula Skew Calibration Algorithm

Calibrating copula skew models involves fitting model parameters to observed tranche prices. The following workflow applies to RFL, Double-t, and other skew models:

**Step 1: Fix marginal survival curves**
- Calibrate single-name survival curves $Q_i(T)$ from CDS market quotes
- These are inputs, not calibration targets

**Step 2: Choose copula family and parameterization**
- RFL: parameters $(\alpha, \beta, \Theta)$ or $(α_1, \alpha_2, \beta, \Theta_1, \Theta_2)$ for two-step
- Double-t: parameters $(\rho, \nu_Z, \nu_\varepsilon)$

**Step 3: Define objective function**
$$\min_\theta \sum_{k} w_k |PV_k(\theta) - PV_k^{\text{market}}|^2$$

where:
- $k$ indexes tranches (0-3%, 3-7%, etc.)
- $w_k$ are weights (often inverse of bid-offer squared)
- $PV_k(\theta)$ is model price given parameters $\theta$
- $PV_k^{\text{market}}$ is observed market price

**Step 4: Numerical optimization**
- Start with reasonable initial guess (from simpler model)
- Use derivative-free optimizer (Nelder-Mead) or gradient-based (BFGS)
- Check multiple starting points for local minima

**Step 5: Validate fit quality**
- Check that all tranches price within bid-offer
- Examine residual pattern (systematic vs. random)
- Test parameter stability under small quote perturbations

**Practical issues:**
1. **Non-convex objective:** Multiple local minima exist; try several starting points
2. **Monte Carlo noise:** If pricing uses Monte Carlo, the objective itself is noisy—use many paths or control variates
3. **Parameter stability:** Small changes in quotes shouldn't cause large parameter jumps—regularization may help
4. **Boundary constraints:** Ensure parameters remain in valid ranges (e.g., $\nu \gt 2$ for finite variance)

> **Desk Reality: Calibration in Practice**
>
> On a correlation desk, calibration happens daily (or intraday). The calibration system must be:
> - Fast enough to run between market opens
> - Stable enough that parameters don't jump wildly
> - Robust enough to handle bad quotes
>
> Many desks maintain "calibration ranges"—if a new parameter falls outside the historical range, it triggers manual review. A parameter that jumps from 30% to 90% overnight usually means bad input data, not a market regime change.

---

## A6.5 Bespoke Tranche Pricing

A **bespoke tranche** is a tranche on a custom portfolio, not the standard index. This section addresses a critical gap: how to price bespoke tranches using the base correlation curve calibrated to index tranches.

### A6.5.1 The Bespoke Problem

O'Kane (Ch. 20.9) frames the challenge: "The issue is that the majority of CDO transactions are on bespoke portfolios with characteristics that can differ significantly from those of the standard indices."

**Key differences between bespoke and index:**
- Different number of credits (could be 50, 75, or 200 vs. 125 for CDX)
- Different spread distribution (higher or lower average spread)
- Different sector composition
- Different recovery assumptions
- Different attachment/detachment points

**The fundamental question:** Given a base correlation curve $\rho_S(K_S)$ calibrated to standard index tranches, what correlation should we use to price a bespoke tranche with strike $K_B$?

### A6.5.2 The Base Correlation Mapping Framework

O'Kane develops a general mapping framework. The goal is to find a function $g$ such that:

$$K_S^{\star} = g(K_B; \mathcal{I}_B, \mathcal{I}_S)$$

where $\mathcal{I}_B$ and $\mathcal{I}_S$ represent information about the bespoke and standard portfolios. Then:

$$\rho_B(K_B) = \rho_S(K_S^{\star})$$

**Desirable properties of the mapping:**
1. Interpolates correctly between known index portfolios
2. Avoids creating arbitrages
3. Captures differences in credit quality and spread dispersion
4. Numerically stable and fast
5. As simple as possible

### A6.5.3 No Mapping

The simplest approach sets $g(x) = x$:

$$K_S^{\star} = K_B$$

**Example (O'Kane):** A bespoke tranche with 4%-8% attachment/detachment simply uses interpolated base correlations $\rho_S(4\\%)$ and $\rho_S(8\\%)$ from the standard curve.

**Problem:** This ignores portfolio differences. Consider pricing a high-yield bespoke using an investment-grade base correlation curve. The 4% equity tranche on a HY portfolio (with, say, 20% expected loss) behaves very differently from a 4% equity tranche on an IG portfolio (with 5% expected loss).

### A6.5.4 At-the-Money (ATM) Correlation Mapping

O'Kane describes ATM mapping (Ahluwalia et al. 2004): the mapping preserves the ratio of tranche strike to portfolio expected loss.

$$\boxed{K_S^{\star} = K_B \times \frac{\mathbb{E}[L_S]}{\mathbb{E}[L_B]}}$$

**Example (O'Kane):** A 4%-8% HY tranche where HY expected loss is 20% and IG expected loss is 5%:

$$K_S^{\star} = 4\\% \times \frac{5\\%}{20\\%} = 1\\%$$

The bespoke 4% strike maps to a 1% standard strike. This makes intuitive sense: a 4% tranche on a risky portfolio should be priced like a more subordinate tranche on a safer portfolio.

**Limitations:**
- Only uses average credit quality (ignores spread dispersion)
- Can map to strikes outside the standard curve range
- May require significant extrapolation

### A6.5.5 Tranche Loss Proportion (TLP) Mapping

The TLP method (O'Kane Ch. 20.9.4) equates the fraction of expected portfolio loss contained in the base tranche:

$$\boxed{\text{TLP}(K) = \frac{\mathbb{E}_{\rho(K)}[\min(L, K)]}{\mathbb{E}[L]}}$$

The mapping finds $K_S^{\star}$ such that:

$$\frac{\mathbb{E}_{\rho_S(K_S^{\star})}[\min(L_S, K_S^{\star})]}{\mathbb{E}[L_S]} = \frac{\mathbb{E}_{\rho_S(K_S^{\star})}[\min(L_B, K_B)]}{\mathbb{E}[L_B]}$$

**Algorithm:**
1. Construct the index TLP curve: for each $K_S$, compute $TLP_S(K_S)$
2. For the bespoke strike $K_B$, compute $TLP_B(K_B)$ for various trial correlations
3. Find $K_S^{\star}$ where the index TLP equals the bespoke TLP
4. Set $\rho_B(K_B) = \rho_S(K_S^{\star})$

**Why TLP is better:** O'Kane explains that TLP captures spread dispersion, not just average spread. A bespoke portfolio with the same average spread but more dispersion will have different TLP behavior.

**Worked Example A6.2: TLP Mapping**

Consider pricing a [5%, 10%] bespoke tranche on a 50-name HY portfolio using CDX IG base correlation.

**Given:**
- CDX IG expected loss: $\mathbb{E}[L_S] = 4\\%$
- Bespoke HY expected loss: $\mathbb{E}[L_B] = 15\\%$
- CDX IG base correlation curve: $\rho_S(3\\%) = 22\\%$, $\rho_S(7\\%) = 35\\%$, etc.

**Step 1:** Compute index TLP at standard strikes:
- At $K_S = 3\\%$: $TLP_S = \mathbb{E}[\min(L_S, 3\\%)] / 4\\% = 2.2\\% / 4\\% = 55\\%$
- At $K_S = 7\\%$: $TLP_S = 3.5\\% / 4\\% = 87.5\\%$

**Step 2:** For bespoke strike $K_B = 5\\%$:
- Using $\rho = 35\\%$: $\mathbb{E}[\min(L_B, 5\\%)] = 4.5\\%$
- $TLP_B(5\\%) = 4.5\\% / 15\\% = 30\\%$

**Step 3:** Find $K_S^{\star}$ where $TLP_S(K_S^{\star}) = 30\\%$.
- Interpolating: $K_S^{\star} \approx 1.5\\%$

**Step 4:** Use $\rho_S(1.5\\%)$ for the bespoke 5% base tranche

### A6.5.6 Model Risk in Bespoke Pricing

O'Kane provides an important warning: "There is no sound theoretical basis for discriminating among these approaches."

**Model risk manifestations:**
- Different mapping methods give different prices
- No method is arbitrage-free in general
- Results depend heavily on how far the bespoke differs from the index

**Best practices:**
1. Use multiple mapping methods and compare
2. Apply wider bid-offer for bespoke vs. index
3. Be conservative on thinly-traded structures
4. Stress test the mapping under different correlation scenarios

> **Desk Reality: Pricing a Bespoke**
>
> When a salesperson asks for a bespoke tranche price, the quant desk typically:
> 1. Runs TLP mapping to get a base price
> 2. Runs ATM mapping as a sanity check
> 3. Compares to a full Monte Carlo with bottom-up calibration
> 4. Adds model risk reserves (often 1-2 points on mezzanine, more on senior)
>
> The "model risk" charge isn't a hedge—it's acknowledgment that no one really knows the right price. Bespoke tranches are quoted wider than index tranches precisely because of this uncertainty.

---

## A6.6 Dynamic Models

### A6.6.1 The Intensity Gamma Model

O'Kane provides extensive treatment of the Intensity Gamma (IG) model of Joshi and Stacey (2005). The key idea is to introduce a time-changed process that generates default clustering.

**The Gamma process driver:** The model uses a gamma process $\Gamma(t)$ to represent "business time"—the arrival of information. A gamma process with shape parameter $\gamma$ and scale parameter $\lambda$ has:
- Increments: $\Gamma(t+s) - \Gamma(t) \sim \text{Gamma}(\gamma s, \lambda)$
- Expected value: $\mathbb{E}[\Gamma(t)] = \gamma t / \lambda$
- Variance: $\text{Var}(\Gamma(t)) = \gamma t / \lambda^2$

**Default mechanism:** Credit $i$ defaults when $\Gamma(t)$ exceeds a random threshold $\theta_i$:
$$\tau_i = \inf\\{t: \Gamma(t) \geq \theta_i\\}$$

where $\theta_i$ is calibrated to match the marginal survival curve:
$$\mathbb{P}(\theta_i \gt x) = Q_i(t) \text{ when } \mathbb{E}[\Gamma(t)] = x$$

**Why this generates clustering:** Large jumps in $\Gamma(t)$ can cause multiple thresholds to be exceeded simultaneously, generating clustered defaults. O'Kane demonstrates this with simulation: "The intensity gamma model exhibits default clustering... we see that defaults do not occur in isolation but tend to occur in groups."

**Calibration:** The model has $2n$ parameters if $n$ gamma processes are used. For a single gamma process:
$$\Gamma(t) = \Gamma_1(t) + \Gamma_2(t)$$

O'Kane reports calibration to CDX NA IG tranches with parameters: $\gamma_1 = 0.0008$, $\gamma_2 = 0.217$, $\lambda_1 = 0.0011$, $\lambda_2 = 0.186$.

### A6.6.2 The Affine Jump Diffusion Model

The AJD model of Duffie and Garleanu (1999) models default intensity dynamics directly:

$$dX(t) = \kappa(\theta - X(t))dt + \sigma\sqrt{X(t)}dW(t) + J \\, dN(t)$$

where:
- $\kappa(\theta - X)$ is mean-reversion
- $\sigma\sqrt{X}dW$ is diffusive volatility (CIR-type)
- $J \\, dN$ is a jump process with intensity $\ell$ and jump size distribution

**Credit-specific intensities:** Each credit $i$ has intensity:
$$\lambda_i(t) = \alpha_i + \beta_i X(t)$$

where $\alpha_i$ is idiosyncratic and $\beta_i X(t)$ is systematic exposure.

**Common vs. idiosyncratic jumps:** The jump process can be decomposed:
$$J \\, dN = J_c \\, dN_c + J_i \\, dN_i$$

where $N_c$ is a common jump process affecting all credits.

**Conditional independence device:** Conditioning on the integrated common factor $Z(T) = \int_t^T X_c(s) \\, ds$, defaults become conditionally independent with conditional default probabilities $p_i(T \mid Z(T))$ given by a model-specific exponential-affine form. This enables the same factor-conditioning computation strategy as static copula models:
1. Condition on $Z(T)$, obtain conditional default probabilities $p_i(T \mid Z(T))$
2. Build conditional portfolio loss distribution (independent Bernoulli trials)
3. Integrate over the distribution of $Z(T)$

**Analytical tractability:** O'Kane shows that survival probabilities can be computed analytically:
$$\mathbb{E}\left[\exp\left(-\int_0^T X(s)ds\right)\right] = \exp(A(T) + B(T)X(0))$$

where $A(T)$ and $B(T)$ satisfy Riccati ODEs. O'Kane notes that time-dependent parameters do not violate the model's no-arbitrage properties "as it would for a copula model."

**Calibration results:** Mortensen (2006), cited by O'Kane, reports calibration to CDX NA IG with parameters: $\kappa = 0.20$, $\sigma = 0.054$, $\ell = 0.037$, $\mu = 0.067$, $w = 0.93$ (common jump fraction).

### A6.6.3 Contagion Models

Contagion models incorporate the idea that one default increases the default probability of others:

**Davis-Lo "infectious default" model:** Credit $i$ can default either:
1. Idiosyncratically with probability $p_i$
2. By "infection" from credit $j$ with probability $q_{ji}$

The infection probability $q_{ji}$ can depend on:
- Industry relationships (supply chain)
- Financial linkages (counterparty exposure)
- Information effects (shared investors)

**Intensity-based contagion:** Default of credit $j$ causes an upward jump in the intensity of credit $i$:
$$\lambda_i(t^+) = \lambda_i(t^-) + \delta_{ji}$$

where $\delta_{ji}$ is the contagion impact.

McNeil et al. in QRM discuss "default contagion and default dependence," noting that "default dependence can arise through direct contagion mechanisms where the default of one firm increases the default probability of others."

> **About Hawkes processes (optional extension):** Some contagion models can be written as self-exciting (Hawkes-style) intensities. This appendix does not fix a specific Hawkes specification; if you want to include one, specify the exact intensity form (or provide a reference excerpt) and calibration targets so the notation is unambiguous.

### A6.6.4 The Markov Chain Approach

Schönbucher's Markov chain model (2005) directly models the portfolio loss process $L(t)$ as a continuous-time Markov chain on states $\\{0, 1, 2, \ldots, N_C\\}$.

**Generator matrix:** The dynamics are governed by the generator matrix $A(t)$ with elements $a_{ij}(t)$ representing the rate of transition from $i$ to $j$ defaults.

For a single-step model (one default at a time), the generator entries are:
$$a_{i,i}(t) = -a_i(t), \qquad a_{i,i+1}(t) = a_i(t), \qquad a_{i,j}(t) = 0 \text{ for } j \notin \{i, i+1\}.$$

**No-arbitrage conditions:** The transition rates must satisfy:
- $a_{ij}(t) \geq 0$ for $i \lt j$
- $a_{ij}(t) = 0$ for $i \gt j$ (no "un-defaults")
- Consistency with observed tranche prices

**Calibration:** O'Kane demonstrates calibration to CDX NA IG tranches by solving for the vector $(a_0, a_1, \ldots, a_{124})$. With 125 parameters and 5 tranches plus the index, exact calibration is possible with many degrees of freedom.

**Stochastic generator extension:** For pricing forward-starting products, the generator itself can be made stochastic:
$$da_n(t) = \mu_n(t)dt + \sigma_n(t)dW_n(t)$$

This allows modeling of spread dynamics and MTM risk.

---

## A6.7 Vasicek Model and Regulatory Applications

### A6.7.1 The Vasicek WCDR Formula

Hull in *Risk Management and Financial Institutions* develops the Vasicek model for credit portfolio risk, which underlies the Basel II/III internal ratings-based (IRB) approach.

For a large homogeneous portfolio with one-year default probability $p$ and asset correlation $\rho$, the **Worst Case Default Rate** at confidence level $X$ is:

$$\boxed{\text{WCDR}(T, X) = \Phi\left(\frac{\Phi^{-1}(p) + \sqrt{\rho}\Phi^{-1}(X)}{\sqrt{1-\rho}}\right)}$$

Hull explains: "This is a strange-looking result, but a very important one. It was first developed by Vasicek in 1987."

**Intuition:** At the $X$-th percentile of the systematic factor distribution, the conditional default probability is WCDR. This is the default rate that will not be exceeded with probability $X$.

**Worked Example A6.3: WCDR Calculation (Hull Example 11.2)**
- Portfolio: Large number of retail loans
- One-year PD: $p = 2\\%$
- Asset correlation: $\rho = 0.10$
- Confidence level: $X = 99.9\\%$

$$\text{WCDR}(1, 0.999) = \Phi\left(\frac{\Phi^{-1}(0.02) + \sqrt{0.10} \times \Phi^{-1}(0.999)}{\sqrt{1-0.10}}\right)$$
$$= \Phi\left(\frac{-2.054 + 0.316 \times 3.090}{0.949}\right)$$
$$= \Phi(0.863) = 0.128$$

The 99.9% worst-case one-year default rate is **12.8%**, dramatically higher than the 2% expected default rate.

### A6.7.2 Basel II/III IRB Capital Formula

The Internal Ratings-Based (IRB) approach uses a modified Vasicek formula for regulatory capital:

$$\boxed{K = \text{LGD} \times \left[\text{WCDR}(1, 0.999) - \text{PD}\right] \times \text{MA}}$$

where:
- LGD is loss-given-default
- MA is a maturity adjustment factor
- The subtraction of PD accounts for expected loss (covered by provisions)
- The 99.9% confidence level is mandated by Basel

**Basel correlation function:** Basel specifies correlation as a function of PD:

$$\rho = 0.12 \times \frac{1 - e^{-50 \times \text{PD}}}{1 - e^{-50}} + 0.24 \times \left(1 - \frac{1 - e^{-50 \times \text{PD}}}{1 - e^{-50}}\right)$$

This gives $\rho \approx 0.24$ for very low PDs and $\rho \approx 0.12$ for high PDs—recognizing that high-PD obligors have more idiosyncratic risk.

**Worked Example A6.4: Basel IRB Capital Calculation**

Calculate IRB capital for a corporate loan portfolio:
- Notional: $100mm
- PD: 2%
- LGD: 45%
- Maturity: 3 years
- Asset correlation (from Basel formula): $\rho = 0.20$

**Step 1: WCDR at 99.9%**
$$\text{WCDR} = \Phi\left(\frac{\Phi^{-1}(0.02) + \sqrt{0.20} \times \Phi^{-1}(0.999)}{\sqrt{0.80}}\right)$$
$$= \Phi\left(\frac{-2.054 + 0.447 \times 3.090}{0.894}\right) = \Phi(0.25) = 0.599$$

Wait—this seems too high. Let me recalculate:
$$= \Phi\left(\frac{-2.054 + 1.382}{0.894}\right) = \Phi(-0.75) = 0.227$$

So WCDR = 22.7%.

**Step 2: Maturity adjustment**
The maturity adjustment formula is:
$$\text{MA} = \frac{1 + (M - 2.5) \times b}{1 - 1.5 \times b}$$

where $b = (0.11852 - 0.05478 \times \ln(\text{PD}))^2 \approx 0.15$ for PD = 2%.

For $M = 3$: MA $\approx 1.06$

**Step 3: Capital requirement**
$$K = 0.45 \times (0.227 - 0.02) \times 1.06 = 0.099 = 9.9\\%$$

**Capital amount:** $100mm × 9.9% = **$9.9mm**

> **Desk Reality: Why Regulatory Capital Matters for Trading**
>
> Regulatory capital requirements affect trading P&L directly:
> - **Return on capital:** A trade that earns 10bp on notional but requires 10% capital has only 100bp ROC
> - **Capital velocity:** Trades that free up capital quickly (short-dated, netting) are more attractive
> - **Balance sheet constraints:** Dealers have capital budgets; expensive trades get wider bid-offers
>
> Correlation desks must understand IRB capital because their hedges consume (or release) capital. A "perfect" hedge that increases capital requirements may not be optimal.

### A6.7.3 Risk-Neutral vs. Physical Calibration

A critical distinction often confused in practice:

| Aspect | Pricing (Risk-Neutral) | Risk (Physical) |
|--------|------------------------|-----------------|
| **Calibration Target** | Tranche quotes | Historical default data |
| **Measure** | Risk-neutral $\mathbb{Q}$ | Physical $\mathbb{P}$ |
| **Use Case** | Mark-to-market, trading | Credit VaR, economic capital |
| **Correlation Source** | Implied from tranches | Estimated from equity/spread correlations |

**Why this matters:** A model calibrated to tranche prices may give different tail probabilities than one calibrated to historical defaults. The risk-neutral measure embeds market risk premia; the physical measure reflects actual default frequencies.

O'Kane emphasizes this distinction throughout, and QRM notes the "data limitations" in physical calibration.

---

## A6.8 Calibration and Validation

### A6.8.1 Calibration Targets

Credit portfolio models can be calibrated to:

| Target | Information Content | Availability |
|--------|---------------------|--------------|
| Single-name CDS | Marginal default probabilities | Liquid, daily |
| Index tranches | Joint loss distribution at traded strikes | Liquid, daily |
| Bespoke tranches | Information at specific strikes | Illiquid |
| Historical defaults | Real-world default clustering | Long history needed |
| Equity correlations | Proxy for asset correlations | Requires model assumptions |

### A6.8.2 Calibration Approaches

**Sequential calibration:**
1. Calibrate marginals to single-name CDS
2. Calibrate dependence to tranche prices
3. Validate on held-out tranches

**Joint calibration:**
- Minimize pricing error across all tranches simultaneously
- Requires optimization over model parameters

**Bootstrap approaches:**
- For Markov chain models, solve for transition rates strike-by-strike
- Similar to curve bootstrapping in rates

### A6.8.3 Model Comparison and Selection

McNeil et al. emphasize "model risk issues" in credit portfolio modeling. Key considerations:

**In-sample fit:** Does the model price traded tranches accurately?

**Out-of-sample prediction:** Does the model price non-traded tranches consistently?

**Parameter stability:** Do calibrated parameters remain stable through time?

**Economic reasonableness:** Are implied correlations and intensities plausible?

**Hedge performance:** Do delta hedges computed from the model work in practice?

### A6.8.4 Validation Tests

**Historical backtesting:**
- Compare model-predicted loss distributions to realized losses
- Challenge: few large portfolios with long default histories

**Stress testing:**
- Evaluate model behavior under extreme scenarios
- Check for numerical instabilities

**Cross-validation:**
- Calibrate to subset of tranches
- Validate on held-out tranches

---

## A6.9 Numerical Implementation

### A6.9.1 Loss Distribution Computation

**Exact recursion (O'Kane Chapter 18):**

For heterogeneous portfolios, the conditional loss distribution is computed iteratively:

1. Start with $f^{(0)}(k) = \mathbf{1}_{\\{k=0\\}}$ (no credits added)
2. For each credit $j = 1, \ldots, N_C$:
   $$f^{(j)}(k) = f^{(j-1)}(k)(1 - p_j(T \mid X)) + f^{(j-1)}(k-1)p_j(T \mid X)$$
3. Integrate over the factor: $f(k) = \int f^{(N_C)}(k \mid X)\phi(X)dX$

**Computational complexity:** $O(N_C^2 \times N_X)$ where $N_X$ is the number of factor quadrature points.

**Efficiency improvement:** Only compute up to the strike of interest: $\min(g, j)$ where $g = \lceil K_2 / u \rceil$.

### A6.9.2 Panjer Recursion

McNeil et al. (QRM Section 10.2.3) describe the Panjer recursion for computing loss distributions in compound models like CreditRisk+:

For a compound distribution where the number of defaults $N$ satisfies $P(N=n) = (a + b/n)P(N=n-1)$ (Panjer class), the loss distribution has the recursion:

$$P(L = k) = \frac{1}{1 - a f_0} \sum_{j=1}^{k} \left(a + \frac{bj}{k}\right) f_j P(L = k-j)$$

where $f_j = P(\text{individual loss} = j)$.

**When to use Panjer:**
- Exact for Poisson, negative binomial, binomial claim counts
- Faster than Monte Carlo for these model classes
- Requires discrete loss grid

### A6.9.3 Approximations for Large Portfolios

**Large Homogeneous Portfolio (LHP):**
$$F_L(\ell) = \Phi\left(\frac{\sqrt{1-\rho}\Phi^{-1}(\ell/(1-R)) - \Phi^{-1}(p)}{\sqrt{\rho}}\right)$$

**Gaussian approximation (Shelton 2004):**
- Approximate conditional loss distribution by Gaussian with matched mean and variance
- Closed-form tranche expected loss

**Adjusted binomial (Andersen, Sidenius, Basu 2003):**
- Match first three moments to a shifted binomial
- More accurate than pure Gaussian for finite portfolios

O'Kane compares these methods for CDX and CDX HY portfolios, finding that the exact recursion is necessary for accuracy at equity strikes but approximations suffice for senior tranches.

**Computing tranche ETL under LHP (practical recipe):** even if you don’t have a closed-form ETL, you can compute it from the LHP loss CDF via 1D integration:

- Base tranche function:
  $$\psi(K) := \mathbb{E}[\min(L,K)] = \int_{0}^{K} \mathbb{P}(L \gt \ell)\\,d\ell = \int_{0}^{K} \left(1 - F_L(\ell)\right)\\,d\ell.$$
- Tranche ETL on $[K_1, K_2]$:
  $$ETL_{[K_1,K_2]} = \frac{\psi(K_2)-\psi(K_1)}{K_2-K_1}.$$

This gives a stable, implementation-friendly route: evaluate $F_L(\ell)$ on a grid and integrate numerically (trapezoid/Simpson).

### A6.9.4 Monte Carlo Methods

For dynamic models, Monte Carlo simulation is often necessary:

**Algorithm for intensity-based models:**
1. Simulate factor paths $X(t)$ over $[0, T]$
2. For each credit, compute integrated intensity: $\Lambda_i(T) = \int_0^T \lambda_i(t)dt$
3. Default if $\Lambda_i(\tau_i) = E_i$ where $E_i \sim \text{Exp}(1)$
4. Aggregate portfolio loss
5. Compute tranche losses and discount

**Variance reduction:**
- Importance sampling (focus on tail events)
- Control variates (use analytical LHP as baseline)
- Stratification on the systematic factor

### A6.9.5 Loss Distribution Comparison: A Visual Guide

Three loss distributions with the same expected loss but different dependence:

| Model | Loss Distribution Shape | Senior Tranche Risk |
|-------|------------------------|---------------------|
| Independent | Tight, concentrated around mean | Very low |
| Gaussian copula | Moderate spread, light tails | Moderate |
| t-copula (low $\nu$) | Wide spread, heavy tails | High |

**Key insight:** Same expected loss, very different tail risk. The choice of copula has enormous implications for senior tranche pricing.

---

## A6.10 Math Sketches: Step-by-Step Derivation Pipelines

This section collects step-by-step derivation outlines with unit checks for the key computational pipelines in credit portfolio modeling. These sketches complement the detailed treatments in earlier sections by providing a unified "cheat sheet" of the core computations.

### A6.10.1 The Universal Pipeline: From Joint Default Model to L(T)

1. **Specify marginal default behavior** for each name at horizon $T$:
$$p_i(T) = \Pr(\tau_i \leq T) = 1 - Q_i(0, T)$$

2. **Specify dependence:**
   - Copula $C$ for $(U_1, \ldots, U_N)$ where $U_i := p_i(\tau_i)$ are uniforms under continuous margins (Sklar), or
   - Factor/intensity dynamics that imply a joint law for $(\tau_i)$

3. **Map defaults to portfolio loss** $L(T)$:
$$L(T) = \sum_{i=1}^{N} w_i (1 - R_i) \mathbf{1}_{\\{\tau_i \leq T\\}}$$

4. **Compute tranche quantities** from $L(T)$ via $TL_{[K_1, K_2]}(L(T))$

**Unit Checks:**
- $w_i$: dimensionless (fraction of notional)
- $(1 - R_i)$: dimensionless
- Indicator: dimensionless
- Hence $L(T)$ dimensionless; consistent with $K_1, K_2$ and tranche widths

**Sanity Bounds:**
- If $\sum_i w_i \leq 1$, then $0 \leq L(T) \leq \sum_i w_i(1 - R_i) \leq 1$
- Tranche loss mapping ensures $0 \leq TL_{[K_1, K_2]}(L) \leq 1$ for any $L \in [0,1]$

### A6.10.2 Static Copula: Limiting Cases

**Independence copula:**
$$C_{\text{ID}}(u_1, \ldots, u_N) = \prod_{i=1}^{N} u_i$$

**Perfect positive dependence (Fréchet-Hoeffding upper bound):**
$$C_M(u_1, \ldots, u_N) = \min(u_1, \ldots, u_N)$$

**Independence limit:** Joint default probabilities factorize; extreme-loss probabilities decay rapidly with $N$ (binomial-like).

**Perfect dependence limit:** Defaults are comonotonic; large jumps in $L(T)$ become likely; senior-tranche ETL rises sharply.

### A6.10.3 Tail Dependence: Gaussian vs. t Copula (Unit Check)

**Gaussian copula:** $\lambda = 0$ when $\rho \lt 1$ (asymptotic tail independence). QRM proves this rigorously.

**$t$ copula:** $\lambda = 2 t_{\nu+1}\\!\left(-\sqrt{\frac{(\nu+1)(1-\rho)}{1+\rho}}\right)$ and it is strictly positive for $\rho \gt -1$.

**Unit check:** $\lambda$ is a probability in $[0,1]$. ✓

### A6.10.4 Factor/Threshold Models: Limiting Cases

**If factor loading → 0:** Conditional PD becomes unconditional; defaults tend to independence.

**If factor loading → 1:** Defaults become highly dependent; tranche losses approach "single systematic driver" behavior.

**Mixture / LT-Archimedean:** QRM shows that varying copula parameter (e.g., Clayton $\theta$) moves from independence ($\theta \to 0$) to comonotonicity ($\theta \to \infty$).

### A6.10.5 Dynamic Bottom-Up Intensities: Unit Checks

**Intensity Gamma (Business-Time) Model:**
- $c_i(t)$: hazard per unit business time (dimension `1/(business-time)`)
- $I(t)$: business time (same dimension as time if scaled)
- $\int c_i \\, dI$ is dimensionless ⇒ exponent is dimensionless ⇒ survival probability is valid ✓

**AJD Correlated Intensities:**
- $\lambda_i(t)$: intensity (units $1/\text{year}$)
- $Z(T) = \int X_c ds$: dimensionless if $X_c$ has units $1/\text{year}$ ✓

### A6.10.6 Top-Down Markov Chain: Sanity Checks

- Rates $a_n \geq 0$ (units $1/\text{year}$) ✓
- If a critical $a_n = 0$, loss support can be artificially capped (O'Kane's caution)
- Calibration determines $A(T)$ so the induced distribution of $L(T)$ reprices index and tranches

---

## A6.11 Measurement and Stress Testing

### A6.11.1 What Risks These Models Target

**Tail risk / clustering risk:** Default dependence "drastically" impacts the right tail of a credit loss distribution; dependence lengthens the upper tail.

**Dispersion risk (distribution-shape risk):** Two models can match the same expected loss (or a single tranche) but imply different tail probabilities.

**Contagion risk:** QRM explicitly discusses default contagion and interacting intensities as an explicit modeling route.

### A6.11.2 Stress Testing Dependence: How to "Shock"

**Static copula shocks:**
- Shock copula parameters (e.g., $\rho$, $\nu$ for $t$ copula; Archimedean $\theta$)
- Compare tail metrics: $\Pr(L(T) \gt x)$, $ETL_{[K_1, K_2]}(T)$

**Factor model shocks:**
- Shock systematic factor realization $F$ (scenario analysis) or increase factor loading(s) $a_i$ to raise correlation

**Intensity shocks (dynamic):**
- *IG:* Shock jump intensity/size parameters of business time $I(t)$; observe survival and ETL
- *AJD:* Shock common jump parameters of $X_c(t)$ to amplify clustering

**Contagion shocks:**

> **Interacting intensities (specification-dependent):** the exact parametric form depends on the chosen interacting-intensity model. For stress testing, a simple schematic is to apply a post-default jump $\\lambda_i(t^+) = \\lambda_i(t^-) + \\delta_{ji}$ to surviving names’ intensities. If you need a specific parametric form (to calibrate, not just to stress), provide the exact model equations/convention you want to use.

### A6.11.3 Sensitivity Outputs

**Tranche PV sensitivity to dependence parameters:**

If closed-form Greeks are not provided, define sensitivity by finite difference:

$$\frac{\partial \text{ETL}}{\partial \theta} \approx \frac{\text{ETL}(\theta + \Delta) - \text{ETL}(\theta - \Delta)}{2\Delta}$$

**For hedging correlations in practice:** O'Kane emphasizes that for bespoke tranches, correlation sensitivity should be measured relative to the index base correlation curve (hedging in standard tranches).

---

## A6.12 Practical Notes

### A6.12.1 Model Risk in Credit Portfolios

**The model risk is large.** O'Kane emphasizes that correlation products "have some risk which we are not able to hedge using the range of available market securities." This unhedgeable model risk requires:
- Conservative reserves
- Stress testing across models
- Transparency about model limitations

**The 2008 lesson:** Base correlation's failures were foreseeable from its theoretical limitations. Model users must understand not just how to use a model, but *when it will fail*.

### A6.12.2 Correlation Desk P&L Drivers

Understanding what moves correlation book P&L:

| Driver | Effect on Equity | Effect on Senior |
|--------|------------------|------------------|
| Index spread widens | Loses (more defaults expected) | Loses (more defaults reach senior) |
| Correlation increases | **Gains** (fewer defaults in equity) | **Loses** (more clustering in tail) |
| Single-name default | Loses notional | Minimal unless attachment breached |
| Index roll | Roll cost/gain | Roll cost/gain |

> **Desk Reality: Hedging a Tranche Position**
>
> A correlation trader holding equity protection typically hedges with:
> 1. **Delta hedge:** Buy index protection (or sell equity to index)
> 2. **Correlation hedge:** Trade other tranches to neutralize correlation sensitivity
> 3. **Jump-to-default:** Buy single-name CDS on high-spread names
>
> The model tells you the hedge ratios—but the model is wrong. Real hedging involves:
> - Running multiple models and taking average or conservative hedges
> - Reserving for model uncertainty
> - Monitoring hedge performance and adjusting

### A6.12.3 Regulatory Implications

**IRB capital:** Banks using internal models for regulatory capital face constraints:
- Correlation functions are prescribed (limited flexibility)
- Model validation requirements
- Conservative parameter choices

**Stress testing:** Post-2008 regulations require stress testing of correlation assumptions. Models should be evaluated at:
- Historical stress correlations
- Expert-specified adverse scenarios
- Reverse stress tests ("what breaks the book?")

### A6.12.4 Implementation Pitfalls

**Numerical precision:** Tranche expected losses at extreme strikes require careful numerics:
- Quadrature for factor integration (50+ points for accuracy)
- Recursion stability at large $N_C$
- Avoiding numerical underflow in tail probabilities

**Calibration stability:** Dynamic models have many parameters:
- Risk of overfitting
- Multiple local minima in optimization
- Need for regularization or parameter constraints

**Hedging mismatches:** Model hedges assume the model is correct:
- Gamma hedging correlation exposure requires rebalancing
- Model changes create "model P&L" distinct from market P&L

---

## A6.13 Worked Examples

### Toy Examples: Building Intuition

The following toy examples use simple numeric parameters to build intuition about how dependence, tail behavior, and model choices affect portfolio credit risk. They complement the more realistic examples that follow.

### Example A6.5: Two-Name Toy — Independent vs. Perfectly Dependent

Let each name have one-year default probability $p = 0.10$.

**Independent defaults:**

| $K$ (defaults) | Probability |
|-----|-------------|
| 0 | $(1-p)^2 = 0.81$ |
| 1 | $2p(1-p) = 0.18$ |
| 2 | $p^2 = 0.01$ |

**Perfect positive dependence (comonotonic):**

| $K$ (defaults) | Probability |
|-----|-------------|
| 0 | $1 - p = 0.90$ |
| 1 | $0$ |
| 2 | $p = 0.10$ |

**Takeaway:** Same marginal PD, but the extreme outcome $K = 2$ jumps from 1% to 10%—a factor of 10.

### Example A6.6: N-Name Toy — Dependence and Extreme Losses

Let $N = 10$, each name with $p = 0.05$.

**Independent defaults:** $K \sim \text{Binomial}(10, 0.05)$.

$$P(K = 5) = \binom{10}{5} 0.05^5 \cdot 0.95^5 = 252 \times 3.125 \times 10^{-7} \times 0.774 \approx 6.09 \times 10^{-5}$$

Summing through $K = 10$: $\Pr(K \geq 5) \approx 6.37 \times 10^{-5} = 0.00637\\%$

**Perfect dependence (comonotonic, identical PDs):**
- $\Pr(K = 10) = p = 0.05$, $\Pr(K = 0) = 0.95$
- $\Pr(K \geq 5) = 5\\%$

**Takeaway:** A dependence shift can move extreme-loss probability by ~three orders of magnitude.

### Example A6.7: Tranche ETL Under Two Dependence Settings

**Tranches:** equity $[0, 3\\%]$ and senior mezz $[10\\%, 15\\%]$.

**Case A (lighter tail):**

| $L$ | Probability |
|-----|-------------|
| 0 | 0.70 |
| 0.02 | 0.20 |
| 0.06 | 0.08 |
| 0.12 | 0.015 |
| 0.25 | 0.005 |

Equity $[0, 3\\%]$ ETL: $0.20 \times 0.6667 + (0.08 + 0.015 + 0.005) \times 1 = 0.1333 + 0.10 = \mathbf{0.2333}$

Senior mezz $[10\\%, 15\\%]$ ETL: $0.015 \times 0.4 + 0.005 \times 1 = 0.006 + 0.005 = \mathbf{0.011}$

**Case B (heavier tail / more clustering):**

| $L$ | Probability |
|-----|-------------|
| 0 | 0.68 |
| 0.02 | 0.17 |
| 0.06 | 0.08 |
| 0.12 | 0.04 |
| 0.25 | 0.03 |

Equity ETL: $0.17 \times 0.6667 + (0.08 + 0.04 + 0.03) \times 1 = 0.1133 + 0.15 = \mathbf{0.2633}$

Senior mezz ETL: $0.04 \times 0.4 + 0.03 \times 1 = 0.016 + 0.03 = \mathbf{0.046}$

**Takeaway:** Tail-weight shifts can mildly move equity ETL but strongly reprice senior mezz ETL—the sensitivity amplifies up the capital structure.

### Example A6.8: Contagion Toy — Post-Default PD Step-Up

> **Note:** QRM discusses default contagion and interacting intensities. The exact parametric form is not in the excerpt, so the following is a purely illustrative toy consistent with "default increases others' intensity."

**Setup (4 names A, B, C, D):**
- Baseline: A defaults with probability 0.02. If A defaults, each of B, C, D defaults with probability 0.06; if A survives, B, C, D default with probability 0.02 (conditional independence assumed).

**$\Pr(\geq 2 \text{ total defaults})$:**

**(i) A defaults and $\geq 1$ of B, C, D defaults:**
$0.02 \times (1 - 0.94^3) = 0.02 \times 0.1694 = 0.003389$

**(ii) A survives and $\geq 2$ of B, C, D default (with $p = 0.02$):**
$P(\geq 2) = 1 - 0.98^3 - 3(0.02)(0.98^2) = 0.001184$
Contribution: $0.98 \times 0.001184 = 0.001160$

**Total:** $\Pr(\geq 2) = 0.003389 + 0.001160 = 0.4549\%$

**Baseline (no contagion; all 4 independent with $p = 0.02$):**
$\Pr(\geq 2) = 1 - 0.98^4 - 4(0.02)(0.98^3) = 0.2336\%$

**Takeaway:** Even a simple post-default PD step-up roughly doubles $\Pr(\geq 2)$.

### Example A6.9: Model Risk — Two Models Fit Same Equity ETL but Differ in Tails

**Model A:**

| $L$ | Probability |
|-----|-------------|
| 0 | 0.675 |
| 0.02 | 0.225 |
| 0.12 | 0.10 |

Equity ETL $[0, 3\\%]$: $0.225 \times 0.6667 + 0.10 \times 1 = \mathbf{0.25}$
Tail: $\Pr(L \gt 10\\%) = \mathbf{0.10}$

**Model B:**

| $L$ | Probability |
|-----|-------------|
| 0 | 0.70 |
| 0.02 | 0.15 |
| 0.25 | 0.15 |

Equity ETL $[0, 3\\%]$: $0.15 \times 0.6667 + 0.15 \times 1 = \mathbf{0.25}$
Tail: $\Pr(L \gt 10\\%) = \mathbf{0.15}$

**Takeaway:** Same fitted equity ETL, different tail probability—this is "dispersion/model-risk" in the tail.

### Example A6.10: Stress Test — Shock Dependence and Compute Delta ETL

Using Models A and B from Example A6.9, compute stress impact on the $[10\\%, 15\\%]$ senior tranche.

**Model A:** Only $L = 0.12$ contributes: $\text{TL}(0.12) = (0.12 - 0.10)/0.05 = 0.4$
$\text{ETL} = 0.10 \times 0.4 = \mathbf{0.04}$

**Model B:** Only $L = 0.25$ contributes: $\text{TL}(0.25) = 1$
$\text{ETL} = 0.15 \times 1 = \mathbf{0.15}$

**Stress impact:** $\Delta\text{ETL} = 0.15 - 0.04 = \mathbf{0.11}$

**Interpretation:** Senior risk is far more sensitive to tail-mass changes than equity.

### Example A6.11: Recovery Interaction — Vary Recovery, Compute Tail Changes

**Default count distribution** for $N = 10$, equal weights $w = 0.1$:

| $K$ | Probability |
|-----|-------------|
| 0 | 0.80 |
| 1 | 0.15 |
| 2 | 0.04 |
| 5 | 0.01 |

**Case 1: $R = 40\\%$ ($\text{LGD} = 0.6$):**
- $\mathbb{E}[L] = 0.15(0.06) + 0.04(0.12) + 0.01(0.30) = \mathbf{0.0168}$
- $\Pr(L \gt 10\\%) = 0.05$
- $\mathbb{E}[L \mid L \gt 10\\%] = (0.04 \times 0.12 + 0.01 \times 0.30)/0.05 = \mathbf{0.156}$

**Case 2: $R = 20\\%$ ($\text{LGD} = 0.8$):**
- $\mathbb{E}[L] = 0.15(0.08) + 0.04(0.16) + 0.01(0.40) = \mathbf{0.0224}$
- $\Pr(L \gt 10\\%) = 0.05$ (unchanged)
- $\mathbb{E}[L \mid L \gt 10\\%] = (0.04 \times 0.16 + 0.01 \times 0.40)/0.05 = \mathbf{0.208}$

**Takeaway:** Lower recovery increases expected loss and tail severity even if tail probability is unchanged.

### Example A6.12: Sanity Check — Detect Impossible ETL / Loss Bounds Violations

**Tranche:** $[10\\%, 15\\%]$, so $0 \leq \text{ETL} \leq 1$.

Suppose someone reports (incorrectly) $ETL_{10-15} = 1.20$. This is **impossible** because tranche loss fraction is bounded by 1.

**How to detect:** Unnormalized expected tranche loss (portfolio-notional units) is $W \times ETL \leq W$. Here $W = 0.05$. If $ETL = 1.20$, unnormalized ETL would be $0.06 \gt 0.05$, violating the maximum possible tranche loss.

**Related "arbitrage smell":** O'Kane shows that poor interpolation can generate negative tranchelet spreads and other arbitrage indicators—another form of sanity failure.

### Example A6.13: Calibration Toy — Two-Parameter Solve

> **Note:** In practice, tranche calibration is nonlinear and is typically done via numerical optimization (minimizing PV errors across instruments). The toy linear “pricing response” below is only to illustrate the mechanics of multi-target calibration.

Suppose a model outputs two tranche ETLs:
$$ETL_{0-3}(\theta_1, \theta_2) = 0.20 + 0.10\theta_1 - 0.05\theta_2$$
$$ETL_{10-15}(\theta_1, \theta_2) = 0.01 + 0.03\theta_1 + 0.04\theta_2$$

Given targets $ETL_{0-3} = 0.25$ and $ETL_{10-15} = 0.02$:

From the first: $0.10\theta_1 - 0.05\theta_2 = 0.05 \Rightarrow \theta_1 = 0.5 + 0.5\theta_2$

Substituting: $0.01 + 0.03(0.5 + 0.5\theta_2) + 0.04\theta_2 = 0.02$
$0.025 + 0.055\theta_2 = 0.02 \Rightarrow \theta_2 = -0.0909$, $\theta_1 = 0.4545$

**Takeaway:** Multi-target calibration pins parameters and can yield unintuitive values (here negative $\theta_2$), prompting constraint checks in real implementations.

---

### Full Worked Examples

### Example A6.14: Base Correlation Calibration

**Problem:** Given 5-year CDX NA IG tranche spreads, compute base correlations.

| Tranche | Upfront (%) | Running (bps) |
|---------|-------------|---------------|
| 0-3% | 40.00 | 500 |
| 3-7% | — | 175 |
| 7-10% | — | 50 |
| 10-15% | — | 17 |
| 15-30% | — | 6 |

**Solution:**

1. **0-3% tranche:** Solve for $\rho$ such that PV = 0:
   - Protection leg PV depends on $\psi(5Y, 3\\%)$ under correlation $\rho$
   - Premium leg PV depends on tranche survival curve
   - Root-finding gives $\rho^{\text{base}}(3\\%) \approx 23\\%$

2. **3-7% tranche:** Must first compute $\rho^{\text{base}}(7\\%)$
   - The 3-7% tranche is: $[0-7\\%] - [0-3\\%]$
   - Solve for $\rho^{\text{base}}(7\\%)$ such that the net position prices correctly
   - Result: $\rho^{\text{base}}(7\\%) \approx 38\\%$

3. Continue for higher strikes...

**Final base correlation curve:**

| Strike | Base Correlation |
|--------|------------------|
| 3% | 23% |
| 7% | 38% |
| 10% | 48% |
| 15% | 57% |
| 30% | 74% |

### Example A6.15: Vasicek WCDR Sensitivity

**Problem:** A bank has a portfolio of 1000 commercial loans with:
- Average one-year PD: 1.5%
- Average LGD: 45%
- Asset correlation: $\rho = 0.15$

Calculate the 99.9% VaR for portfolio credit losses.

**Solution:**

1. **WCDR at 99.9%:**

First, compute the intermediate values:
- $\Phi^{-1}(0.015) = -2.170$
- $\sqrt{0.15} = 0.387$
- $\Phi^{-1}(0.999) = 3.090$
- $\sqrt{1 - 0.15} = \sqrt{0.85} = 0.922$

Applying the WCDR formula:
$$\text{WCDR} = \Phi\left(\frac{\Phi^{-1}(0.015) + \sqrt{0.15} \times \Phi^{-1}(0.999)}{\sqrt{0.85}}\right)$$
$$= \Phi\left(\frac{-2.170 + 0.387 \times 3.090}{0.922}\right)$$
$$= \Phi\left(\frac{-2.170 + 1.196}{0.922}\right)$$
$$= \Phi\left(\frac{-0.974}{0.922}\right) = \Phi(-1.056)$$
$$\approx 0.145$$

So **WCDR = 14.5%** at the 99.9% confidence level.

2. **99.9% VaR:**
$$VaR_{0.999} = WCDR \times LGD \times \text{Portfolio Notional}$$

If portfolio notional is \$100 million:
Numerically, `VaR_0.999 = 0.145 * 0.45 * 100 = 6.525` (USD millions).

3. **Expected loss for comparison:**
$$EL = p \times LGD \times \text{Notional} = 0.015 \times 0.45 \times 100 = 0.675$$

The unexpected loss (VaR minus EL) is $6.525 - 0.675 = 5.85$ (USD millions).

### Example A6.16: Intensity Gamma Model Simulation

**Problem:** Simulate defaults under the IG model with parameters $\gamma = 0.1$, $\lambda = 0.5$ for a portfolio of 100 credits with 5-year survival probabilities $Q_i(5) = 0.95$.

**Algorithm:**

1. Generate gamma process at horizon: $\Gamma(5) \sim \text{Gamma}(\gamma \times 5, \lambda) = \text{Gamma}(0.5, 0.5)$
2. For each credit $i$:
   - Draw threshold: $\theta_i = Q_i^{-1}(U_i)$ where $U_i \sim \text{Uniform}(0,1)$
   - Default if $\Gamma(5) \geq \theta_i$
3. Count defaults and compute portfolio loss

**Single path example:**
- Draw $\Gamma(5) = 1.2$
- For credit 1: $U_1 = 0.03$, $\theta_1 = Q^{-1}(0.03) = -\ln(0.97) / \lambda_{\text{credit}} \approx 0.6$
- Since $\Gamma(5) = 1.2 \gt 0.6$, credit 1 defaults
- Repeat for all credits

The simulation exhibits **clustering**: when $\Gamma(5)$ is large, many credits default; when small, few default.

### Example A6.17: Tranche Delta in the Gaussian Copula

**Problem:** Compute the systemic delta (index hedge ratio) for a 0-3% equity tranche under the Gaussian copula.

**Solution:**

The tranche delta measures sensitivity to a parallel shift in all spreads:

$$\Delta_{\text{systemic}} = \frac{\partial V_{\text{tranche}}}{\partial S_{\text{index}}}$$

O'Kane reports that for the 0-3% equity tranche on CDX NA IG:
- Systemic delta (base correlation model): **23.03**
- Systemic delta (intensity gamma model): **24.85**

This means a \$10 million notional equity tranche position should be hedged with approximately \$230 million notional of the index.

**Interpretation:** The equity tranche has high leverage—small index spread moves cause large tranche PV changes. The leverage decreases for more senior tranches.

---

## Summary

This appendix has extended beyond the base correlation framework to survey the landscape of credit portfolio models:

1. **Base correlation limitations:** The framework is a quoting convention, not a consistent pricing model. It fails to conserve expected loss under interpolation, has no spread dynamics, and cannot capture crisis behavior.

2. **Static extensions:** Alternative copulas (t, Clayton), random factor loading (RFL), Double-t copula, and stochastic recovery can address some limitations while preserving tractability. RFL and Double-t specifically target correlation skew fitting.

3. **Bespoke tranche pricing:** TLP mapping and ATM mapping provide methods to extend base correlation to non-standard portfolios, but model risk remains high.

4. **Dynamic intensity models:** The Intensity Gamma and Affine Jump Diffusion models provide explicit dynamics for default clustering and spread evolution.

5. **Top-down models:** Markov chain approaches model the portfolio loss process directly, enabling natural calibration to tranche prices and pricing of forward-starting products.

6. **Regulatory models:** The Vasicek WCDR formula underlies Basel IRB capital requirements, providing a link between portfolio credit models and bank regulation.

7. **Model risk:** All credit portfolio models have significant limitations. Users must understand not just how to use models, but when they will fail.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Portfolio loss distribution** | Probability law of aggregate losses | Drives tranche pricing and risk |
| **Tail dependence** | Probability of joint extreme events | Senior tranche risk; Gaussian copula has none |
| **Base correlation** | Correlation implied by equity tranches | Market quoting convention; not consistent model |
| **Conditional independence** | Independence given systematic factor | Enables tractable computation |
| **WCDR** | Worst Case Default Rate at confidence level | Regulatory capital formula |
| **RFL model** | Random Factor Loading | Fits correlation skew via state-dependent correlation |
| **Double-t copula** | t-distributed factors (both systematic and idiosyncratic) | Fat tails in both dimensions |
| **TLP mapping** | Tranche Loss Proportion | Bespoke pricing method |
| **Dynamic model** | Specifies evolution of intensities/loss | Needed for MTM risk, options |
| **Top-down vs. bottom-up** | Model loss directly vs. aggregate individual | Different calibration and hedging properties |
| **Contagion** | Defaults increase other default probabilities | Crisis propagation mechanism |
| **Intensity gamma** | Time-changed default process | Generates realistic clustering |
| **Markov chain model** | Loss as continuous-time Markov chain | Direct tranche calibration |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Why does the Gaussian copula require higher base correlations for senior tranches? | Zero tail dependence: the model cannot generate sufficient extreme joint default probability, so higher correlation compensates |
| 2 | What is the WCDR formula? | $\text{WCDR}(T, X) = \Phi\left(\frac{\Phi^{-1}(p) + \sqrt{\rho}\Phi^{-1}(X)}{\sqrt{1-\rho}}\right)$ |
| 3 | What is tail dependence? | $\lambda_L = \lim_{u \to 0^+} \mathbb{P}(U_2 \leq u \mid U_1 \leq u)$—the probability of joint extremes |
| 4 | Why does the Gaussian copula have zero tail dependence? | As $u \to 0$, joint probability of both normals being extreme vanishes faster than linear |
| 5 | What is conditional independence in factor models? | Given the systematic factor, defaults are independent Bernoulli trials |
| 6 | What is the key advantage of top-down models? | Direct calibration to tranche prices without modeling individual names |
| 7 | What does the intensity gamma model use to generate clustering? | A gamma process representing "business time" with jumps |
| 8 | What is the AJD model? | Affine Jump Diffusion: intensities follow mean-reverting diffusion with jumps |
| 9 | What is contagion in credit models? | Defaults increase the default probability of surviving credits |
| 10 | Why is base correlation "not a consistent pricing model"? | Interpolation can violate no-arbitrage; different correlations for different tranches |
| 11 | What are the no-arbitrage conditions on expected tranche loss $\psi(T, K)$? | $\partial \psi / \partial K \in [0,1]$ and $\partial^2 \psi / \partial K^2 \leq 0$ |
| 12 | What is the generator matrix in Markov chain models? | Matrix $A(t)$ with $a_{ij}(t)$ = rate of transition from $i$ to $j$ defaults |
| 13 | How does stochastic recovery affect tranche risk? | Low recovery in bad states increases senior tranche risk without changing correlation |
| 14 | What is the factor loading in the one-factor model? | $\sqrt{\rho}$—fraction of latent variable from systematic factor |
| 15 | What does the Basel II correlation function capture? | Higher idiosyncratic risk for high-PD obligors: $\rho$ decreases with PD |
| 16 | Why do dynamic models matter for hedging? | Static models don't capture how spreads move after defaults |
| 17 | What is the recursion for computing loss distribution? | $f^{(j)}(k) = f^{(j-1)}(k)(1-p_j) + f^{(j-1)}(k-1)p_j$ |
| 18 | What is importance sampling used for in credit MC? | Focus simulation on tail events to reduce variance |
| 19 | What is the t-copula tail dependence formula? | $\lambda = 2t_{\nu+1}(-\sqrt{(\nu+1)(1-\rho)/(1+\rho)})$ |
| 20 | What happens to base correlation during a crisis? | Increases dramatically across all strikes |
| 21 | What is the LHP approximation? | Large Homogeneous Portfolio: closed-form loss distribution for identical credits |
| 22 | Why is model risk large in credit portfolios? | Unhedgeable correlation risk; model failures concentrate in tail events |
| 23 | What is the equity tranche's correlation sensitivity? | Long correlation (gains when correlation rises) |
| 24 | What is the senior tranche's correlation sensitivity? | Short correlation (loses when correlation rises) |
| 25 | How many parameters does a single-step Markov chain model have for $N$ credits? | $N$ transition rates $(a_0, a_1, \ldots, a_{N-1})$ |
| 26 | What is the key flaw of Gaussian copula for senior tranches? | Tail independence ($\lambda = 0$) underestimates joint extreme defaults |
| 27 | What does RFL stand for and what does it fix? | Random Factor Loading; allows fitting correlation skew via state-dependent correlation |
| 28 | State the TLP method for bespoke pricing (one sentence). | Map bespoke attachment to index attachment with same expected tranche loss proportion |
| 29 | Basel IRB confidence level for capital calculation? | 99.9% |
| 30 | What is Panjer recursion used for? | Computing loss distribution analytically for compound models (CreditRisk+) |
| 31 | IG model: what creates default clustering? | Common jumps in business time $\Gamma(t)$ |
| 32 | Why is base correlation "not an arbitrage-free model"? | Different $\rho$ for different strikes; not from a single loss distribution |
| 33 | What is the Double-t copula enhancement over standard t? | Both factor AND idiosyncratic components are t-distributed |
| 34 | Key difference: pricing vs. risk calibration? | Pricing uses risk-neutral (implied from quotes); risk uses physical (historical data) |
| 35 | What makes a bespoke tranche "bespoke"? | Custom portfolio, not the standard index |
| 36 | What is the correlation formula in a two-factor model? | $c_{ij} = \beta_{1i}\beta_{1j} + \beta_{2i}\beta_{2j}$ |
| 37 | How many factors are needed to model $M$ sectors? | $M$ factors—one-factor models cannot capture sector structure |
| 38 | What happens to credit B's spread when positively correlated credit A defaults? | B's spread jumps up; the magnitude increases with correlation |
| 39 | What is ETL interpolation? | Interpolate expected tranche loss directly, then invert to base correlation |
| 40 | Why is ETL interpolation preferred over base correlation interpolation? | No-arbitrage constraints ($\partial\psi/\partial K \in [0,1]$, convexity) are easier to enforce in ETL space |
| 41 | What is the PCHIP spline? | Piecewise Cubic Hermite Interpolant: guarantees smoothness and monotonicity |
| 42 | What is the time-dimension no-arbitrage constraint for base correlation surface? | $\partial^2\psi/\partial T\partial K \geq 0$ |
| 43 | What is the Clayton copula's lower tail dependence coefficient? | $\lambda_L = 2^{-1/\theta}$ for $\theta \gt 0$; e.g., $\theta = 2$ gives $\lambda_L \approx 0.71$ |
| 44 | What is CreditRisk+? | A Poisson mixture model with gamma-distributed factors; defaults are conditionally Poisson |
| 45 | What distribution does the number of defaults follow in CreditRisk+? | Sum of independent negative binomial random variables (one per gamma factor) |
| 46 | RFL model: what are the calibrated parameters for CDX NA IG? | $\alpha = 54.25\\%$, $\beta = 31.16\\%$, $\Theta = -2.58$ |
| 47 | Double-t model: what are the calibrated parameters for CDX NA IG? | $\nu_Z = 7.0$, $\nu_\varepsilon = 2.5$, $\rho = 15.0\\%$ |
| 48 | ATM mapping formula for bespoke pricing? | $K_S^{\star} = K_B \times \frac{\mathbb{E}[L_S]}{\mathbb{E}[L_B]}$ |

---

## Mini Problem Set

### Problem 1: Tail Dependence Comparison
Calculate the lower tail dependence coefficient $\lambda_L$ for:
(a) Gaussian copula with $\rho = 0.5$
(b) Student's t-copula with $\rho = 0.5$ and $\nu = 4$ degrees of freedom
(c) Interpret the difference for senior tranche pricing.

**Solution sketch:** (a) $\lambda_L = 0$ for Gaussian. (b) Use $\lambda = 2t_5(-\sqrt{5 \cdot 0.5/1.5}) = 2t_5(-1.29) \approx 0.18$. (c) The t-copula generates 18% probability of joint extreme defaults vs. 0% for Gaussian—critical for senior tranches.

### Problem 2: WCDR Sensitivity
A portfolio has PD = 2% and correlation $\rho = 0.20$.
(a) Compute WCDR at 99.9% confidence
(b) How does WCDR change if $\rho$ increases to 0.30?
(c) How does WCDR change if PD increases to 3% (keeping $\rho = 0.20$)?

**Solution sketch:** (a) WCDR = $\Phi((-2.054 + 0.447 \times 3.09)/0.894) = \Phi(0.25) \approx 0.60$ — wait, let me recalculate: $= \Phi((-2.054 + 1.38)/0.894) = \Phi(-0.75) = 0.23$. (b) Higher $\rho$ increases WCDR. (c) Higher PD increases WCDR.

### Problem 3: Base Correlation Arbitrage
Given base correlations: $\rho^B(3\\%) = 25\\%$, $\rho^B(7\\%) = 40\\%$. Linear interpolation gives $\rho^B(5\\%) = 32.5\\%$.
(a) Compute the expected tranche losses $\psi(5Y, K)$ for $K = 3\\%, 5\\%, 7\\%$
(b) Check whether the convexity condition holds
(c) Describe an arbitrage if it doesn't

**Solution sketch:** Compute ETLs using Gaussian copula. If slope increases (convexity violated), buy 5% base tranche, sell 3% and 7% in butterfly structure.

### Problem 4: RFL Model Interpretation
The RFL model for CDX NA IG has calibrated parameters $\alpha = 54.25\\%$, $\beta = 31.16\\%$, $\Theta = -2.58$.
(a) What is the correlation in the "bad state" ($Z \lt \Theta$)?
(b) What is the probability of being in the "bad state"?
(c) Why does $\alpha \gt \beta$ fit the correlation smile?

**Solution sketch:** (a) Approximately $\alpha^2 \approx 29\\%$. (b) $\Phi(-2.58) \approx 0.5\\%$. (c) Higher correlation in bad states makes senior tranches riskier, requiring higher base correlation to match market.

### Problem 5: Double-t vs. Gaussian
Compare the Double-t model ($\nu_Z = 7$, $\nu_\varepsilon = 2.5$, $\rho = 15\\%$) to Gaussian ($\rho = 30\\%$).
(a) Which has fatter tails in the systematic factor?
(b) Which has more idiosyncratic jumps?
(c) Why might Double-t have lower calibrated correlation?

**Solution sketch:** (a) Double-t (df = 7 vs. $\infty$). (b) Double-t (df = 2.5 is very fat). (c) Fat tails generate dependence even with low correlation.

### Problem 6: TLP Mapping
Use TLP mapping to price a [5%, 10%] bespoke tranche:
- Bespoke portfolio expected loss: 12%
- Index expected loss: 4%
- Index base correlation: $\rho(3\\%) = 22\\%$, $\rho(7\\%) = 35\\%$

(a) Compute the bespoke TLP at $K_B = 5\\%$
(b) Find the index strike $K_S^{\star}$ with the same TLP
(c) What base correlation should be used?

### Problem 7: Basel IRB Capital
Calculate Basel IRB capital for a portfolio:
- PD = 2%, LGD = 45%, EAD = $100mm
- Basel correlation: $\rho = 0.12$ (high-PD corporate)

(a) Compute WCDR at 99.9%
(b) Compute capital as percentage of EAD
(c) How much capital (in $) is required?

**Solution sketch:** (a) WCDR $\approx 15\\%$. (b) K = 0.45 × (0.15 - 0.02) = 5.85%. (c) $5.85mm.

### Problem 8: Markov Chain Generator
For a 100-credit portfolio, the Markov chain model has transition rates $(a_0, a_1, \ldots, a_{99})$.
(a) If $a_n = 0.1$ for all $n$, what is the expected number of defaults by $T = 5$?
(b) If $a_n = 0.1 \times (1 + 0.01n)$, how does this change the loss distribution?
(c) Interpret the economic meaning of increasing transition rates.

### Problem 9: Multi-Factor Model
Extend the one-factor model to two factors (industry $X_1$ and market $X_2$):
$$Z_i = \beta_{i1}X_1 + \beta_{i2}X_2 + \sqrt{1 - \beta_{i1}^2 - \beta_{i2}^2}\varepsilon_i$$

(a) What is the correlation between credits in the same industry? Different industries?
(b) How many parameters are needed for 125 credits in 10 industries?
(c) What information could calibrate the industry-specific loadings?

### Problem 10: Dynamic Model Spread Dynamics
In the AJD model, suppose the common intensity factor $X(t)$ jumps by $\Delta X = 0.5$ due to a credit event.
(a) How does the survival probability of remaining credits change?
(b) How does the expected tranche loss change?
(c) Why does this matter for mark-to-market risk?

### Problem 11: ETL Interpolation
Given base correlations at standard strikes:
- $\rho^B(3\\%) = 22\\%$ → $\psi(5Y, 3\\%) = 1.1\\%$
- $\rho^B(7\\%) = 35\\%$ → $\psi(5Y, 7\\%) = 2.0\\%$
- $\rho^B(10\\%) = 45\\%$ → $\psi(5Y, 10\\%) = 2.5\\%$

(a) Use linear interpolation in ETL space to compute $\psi(5Y, 5\\%)$
(b) Check the convexity condition: is $\partial^2\psi/\partial K^2 \leq 0$?
(c) Compare to linear interpolation in base correlation space

**Solution sketch:** (a) $\psi(5\\%) = 1.1 + (2.0-1.1) \times (5-3)/(7-3) = 1.55\\%$. (b) Slopes: $(2.0-1.1)/4 = 0.225$, $(2.5-2.0)/3 = 0.167$; slope decreasing ✓ convex. (c) Base corr interpolation may violate convexity.

### Problem 12: Correlation Skew Stress Test
Design a stress test that shocks correlation from 0.20 to 0.40.
(a) Compute the change in ETL for [0-3%] equity tranche
(b) Compute the change in ETL for [10-15%] senior tranche
(c) Which tranche gains and which loses from higher correlation?

**Solution sketch:** (a) Higher $\rho$ reduces equity ETL (fewer defaults in equity). (b) Higher $\rho$ increases senior ETL (more tail clustering). (c) Equity gains, senior loses—this is "long correlation" vs. "short correlation."

### Problem 13: Default-Induced Spread Dynamics
Credits A and B have flat 5Y survival probability $Q_A(5) = Q_B(5) = 0.95$ and asset correlation $\rho = 0.40$.

(a) Compute the default thresholds $C_A(5)$ and $C_B(5)$
(b) If A defaults at $t = 0$ (instantaneous), what is B's conditional survival probability to $T = 5$?
(c) How much does B's 5Y spread change after A's default?

**Solution sketch:** (a) $C_i(5) = \Phi^{-1}(0.05) = -1.645$. (b) Use O'Kane formula: conditional probability is lower. (c) Spread increases by roughly 30-50% depending on parameters.

### Problem 14: Panjer Recursion Setup
A CreditRisk+ model has two sectors with gamma factors $\Psi_1 \sim \text{Ga}(4, 4)$ and $\Psi_2 \sim \text{Ga}(9, 9)$.
(a) What are the mean and variance of each factor?
(b) What is the marginal distribution of the number of defaults?
(c) Why is this more tractable than Monte Carlo?

**Solution sketch:** (a) Mean = 1, Var = 1/4 and 1/9. (b) Sum of two negative binomials. (c) Panjer recursion gives exact distribution without simulation.

### Problem 15: Model Risk Quantification
A correlation desk prices the same [3-7%] tranche using three models:
- Base correlation: 250 bps
- RFL: 280 bps
- Double-t: 240 bps

(a) What is the range of model-implied prices?
(b) How might a trader set the bid-offer?
(c) What does this spread tell us about model risk?

**Solution sketch:** (a) 40 bps range. (b) Bid-offer should be at least as wide as model uncertainty, perhaps 30-50 bps. (c) Significant model risk—no single "correct" price.

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (base correlation, tranche pricing conventions, model alternatives, and practical desk framing)
- Alexander J. McNeil, Rüdiger Frey, & Paul Embrechts, *Quantitative Risk Management* (copulas, tail dependence, and credit portfolio loss modeling foundations)
- Philipp J. Schönbucher, *Credit Derivatives Pricing Models* (top-down loss models, Markov chain approaches, and credit derivatives modeling frameworks)
- John C. Hull, *Risk Management and Financial Institutions* (credit risk measurement and regulatory capital context)


*Appendix A6 of Fixed Income: Practice and Theory*
