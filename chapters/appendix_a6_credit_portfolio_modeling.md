# Appendix A6: Credit Portfolio Modeling Beyond Base Correlation

---

## Introduction

Base correlation worked until it didn't. Then what?

In the summer of 2007, the correlation "smile" that structured credit desks had calibrated for years began to exhibit behavior no one-factor Gaussian model could explain. Senior tranches that had always traded at seemingly safe spreads suddenly required base correlations exceeding 90%—mathematically implausible values that revealed the model was being asked to do something it was never designed to do. As O'Kane observes, the base correlation framework "is not a pricing model but rather a quoting convention"—a language for translating tranche prices into a single parameter, not a theory of how defaults actually cluster.

The 2007-2008 crisis exposed three fundamental limitations of the base correlation approach: (1) the single-parameter Gaussian copula cannot capture the *dynamics* of how correlation evolves through a crisis; (2) base correlation interpolation across non-traded strikes creates arbitrage-prone prices; and (3) the model's complete lack of tail dependence systematically underestimates senior tranche risk precisely when that risk materializes. These failures were not surprises to quantitative researchers—many had been documented in academic literature—but the crisis made them operationally urgent.

This appendix extends beyond the base correlation framework introduced in Chapter 50 to survey the broader landscape of credit portfolio models. We begin with the fundamental building blocks—portfolio loss distributions, dependence structures, and the distinction between bottom-up and top-down approaches (Section A6.1). Section A6.2 examines why base correlation "is not enough," providing a rigorous treatment of its empirical failures. Section A6.3 develops a framework map organizing the alternatives: static copula extensions, factor model enhancements, dynamic intensity models, and other mechanisms. Sections A6.4-A6.6 provide detailed treatment of selected advanced models. Section A6.7 addresses calibration and validation—the practical challenge of making these models work. Section A6.8 covers numerical implementation. Throughout, we balance mathematical rigor with practical guidance for the desk quant who must actually implement, calibrate, and risk-manage these models.

**Prerequisites:** This appendix assumes familiarity with tranche mechanics (Chapter 48), expected tranche loss calculations (Chapter 49), and the Gaussian copula framework (Chapter 50). Readers should be comfortable with measure-theoretic probability at the level of Appendix A1.

---

## Conventions and Notation

| Symbol | Definition |
|--------|------------|
| $N_C$ | Number of credits in reference portfolio |
| $\tau_i$ | Default time of credit $i$ |
| $Q_i(T) = \mathbb{P}(\tau_i > T)$ | Survival probability to time $T$ |
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

---

## A6.1 Core Concepts: The Building Blocks

### A6.1.1 Portfolio Loss Distribution

The portfolio loss $L(T)$ at horizon $T$ is the central object of interest. For a portfolio of $N_C$ credits with exposures $w_i$ (summing to 1) and loss-given-defaults $L_i$:

$$\boxed{L(T) = \sum_{i=1}^{N_C} w_i L_i \mathbf{1}_{\{\tau_i \leq T\}}}$$

The **loss distribution** is the probability law of $L(T)$. McNeil, Frey, and Embrechts in *Quantitative Risk Management* emphasize that "the main quantity of interest in portfolio credit risk is the distribution of portfolio losses over a fixed time horizon."

For tranche pricing, we need the expected loss on tranches of the form $[K_1, K_2]$:

$$\mathbb{E}[L(T; K_1, K_2)] = \frac{\mathbb{E}[\min(L(T), K_2)] - \mathbb{E}[\min(L(T), K_1)]}{K_2 - K_1}$$

This depends on the *entire distribution* of $L(T)$, not just its mean.

**Key insight:** Two portfolios with identical expected losses but different loss *distributions* will produce different tranche prices. The challenge is to model the distribution, not just the mean.

### A6.1.2 Dependence Modeling: Why Correlation Is Not Enough

The term "correlation" is used loosely in credit markets, but McNeil et al. clarify the distinction between different dependence concepts:

**Default correlation:** The correlation between default indicators:
$$\rho_{ij}^{\text{def}}(T) = \text{Corr}(\mathbf{1}_{\{\tau_i \leq T\}}, \mathbf{1}_{\{\tau_j \leq T\}})$$

**Asset correlation:** In structural models, the correlation between latent asset values:
$$\rho_{ij}^{\text{asset}} = \text{Corr}(Z_i, Z_j)$$

**Copula correlation:** The correlation parameter in a copula specification.

These are *not* the same. As McNeil et al. note, "linear correlation is not a particularly useful measure of dependence for non-elliptical distributions." Credit applications typically involve highly non-normal (indeed, binary) outcomes where linear correlation captures only a fraction of the dependence structure.

**Tail dependence** is often more relevant for senior tranches. McNeil et al. define the lower tail dependence coefficient as:

$$\lambda_L = \lim_{u \to 0^+} \mathbb{P}(U_2 \leq u \mid U_1 \leq u)$$

where $(U_1, U_2)$ have copula $C$. This measures the probability of joint extreme events in the lower tail—precisely what matters for credit portfolio losses.

**The Gaussian copula's fatal flaw:** McNeil et al. prove that $\lambda_L = 0$ for the Gaussian copula whenever $\rho < 1$. No matter how high the correlation, as we move into the extreme tail, joint defaults appear to occur independently. This is a critical limitation for senior tranches, which are exposed precisely to these tail events.

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

$$\mathbb{P}(\tau_1 > t_1, \ldots, \tau_{N_C} > t_{N_C}) = C(Q_1(t_1), \ldots, Q_{N_C}(t_{N_C}))$$

**Common copulas in credit:**

| Copula | Parameters | Tail Dependence | Use Case |
|--------|------------|-----------------|----------|
| Gaussian | Correlation matrix $\Sigma$ | None ($\lambda_L = \lambda_U = 0$) | Market standard |
| Student's $t$ | $\Sigma$, degrees of freedom $\nu$ | Symmetric: $\lambda_L = \lambda_U > 0$ | Fat-tail modeling |
| Clayton | $\theta > 0$ | Lower only: $\lambda_L = 2^{-1/\theta}$ | Asymmetric downside |
| Gumbel | $\theta > 1$ | Upper only: $\lambda_U = 2 - 2^{1/\theta}$ | Asymmetric upside |
| Archimedean | Generator $\phi$ | Varies | Flexible structures |

**Clayton copula tail dependence:** McNeil et al. (QRM, Example 5.31) derive the lower tail dependence coefficient for the Clayton copula:

$$\boxed{\lambda_L^{\text{Clayton}} = 2^{-1/\theta} \quad \text{for } \theta > 0}$$

For $\theta = 2$, this gives $\lambda_L = 2^{-0.5} \approx 0.71$—substantial tail dependence even with moderate parameter values. The Clayton copula has *no* upper tail dependence ($\lambda_U = 0$), making it appropriate for modeling situations where joint extreme *losses* (lower tail) are more likely than joint extreme *gains*. This asymmetry can be valuable for credit portfolios where crisis contagion is a primary concern.

### A6.1.4 Factor Models

Factor models reduce the dimensionality of dependence by assuming that default correlation arises through exposure to common factors. The canonical one-factor model (Chapter 50) has:

$$Z_i = \sqrt{\rho} X + \sqrt{1-\rho} \varepsilon_i$$

where $X$ is the systematic factor and $\varepsilon_i$ are idiosyncratic shocks.

**Multi-factor extensions** allow for richer correlation structures:

$$Z_i = \sum_{k=1}^{K} \beta_{ik} X_k + \sqrt{1 - \sum_{k=1}^{K} \beta_{ik}^2} \varepsilon_i$$

where $X_k$ might represent industry, region, or rating factors. For a two-factor model, the correlation between credits $i$ and $j$ is:

$$\boxed{c_{ij} = \beta_{1i}\beta_{1j} + \beta_{2i}\beta_{2j}}$$

**Sector correlation example (O'Kane):** Consider four credits grouped into two sectors (A and B, two credits each). With a two-factor model where:
- All credits have equal exposure to the first (market) factor: $\beta_{1a} = \beta_{1b} = 0.6$
- Sector A credits are negatively correlated to the second factor: $\beta_{2a} = -0.4$
- Sector B credits are positively correlated to the second factor: $\beta_{2b} = 0.5$

The resulting correlation matrix is:

$$C = \begin{pmatrix}
& \text{Sector A} & & \text{Sector B} & \\
100\% & 52\% & | & 16\% & 16\% \\
52\% & 100\% & | & 16\% & 16\% \\
\hline
16\% & 16\% & | & 100\% & 61\% \\
16\% & 16\% & | & 61\% & 100\%
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
- Aggregate to portfolio loss: $L(T) = \sum_i w_i L_i \mathbf{1}_{\{\tau_i \leq T\}}$
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

### A6.1.7 Default-Induced Spread Dynamics in the Latent Variable Model

Although the latent variable model is static, it implies *implicit* spread dynamics when we condition on observed defaults. O'Kane derives the conditional survival curve for credit B given that credit A defaults at time $t$:

$$\boxed{\hat{Q}_B(t, T) = 1 - \frac{\Phi\left(\frac{C_B(T) - \rho C_A(t)}{\sqrt{1-\rho^2}}\right) - \Phi\left(\frac{C_B(t) - \rho C_A(t)}{\sqrt{1-\rho^2}}\right)}{1 - \Phi\left(\frac{C_B(t) - \rho C_A(t)}{\sqrt{1-\rho^2}}\right)}}$$

where $C_i(T) = \Phi^{-1}(1 - Q_i(T))$ is the default threshold.

**Key observations from O'Kane:**

1. **Zero correlation:** When $\rho = 0$, we have $\hat{Q}_B(t,T) = Q_B(t,T)$—the default of A has no impact on B's survival curve. "This makes sense: if there is no correlation between A and B, the spread curve of credit B should not change after a default of credit A."

2. **Positive correlation:** The conditional spread curve jumps *up* when A defaults. The magnitude increases with $\rho$.

3. **Negative correlation:** The conditional spread curve jumps *down*—B becomes safer when A defaults.

4. **Time decay:** "The size of the spread jump declines with $t = \tau_A$, i.e., how far in the future credit A defaults." This reveals non-stationary behavior in the Gaussian model.

**Implication for hedging:** This implicit spread dynamic affects how the portfolio responds after defaults. A proper dynamic model would make these effects explicit and hedgeable.

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

### A6.2.2 Arbitrage and Interpolation Pathologies

O'Kane identifies a fundamental problem: "base correlation is not a consistent pricing framework."

**The interpolation problem:** Base correlations are only observable at traded strikes (e.g., 3%, 7%, 10%, 15%, 30%). To price bespoke tranches with different strikes, one must interpolate. But:

1. **Linear interpolation in base correlation space** can violate no-arbitrage constraints
2. **No-arbitrage interpolation** requires constraints on the expected tranche loss function $\psi(T, K)$

The no-arbitrage conditions from O'Kane (Chapter 19) require:

$$\frac{\partial \psi(T, K)}{\partial K} \in [0, 1]$$
$$\frac{\partial^2 \psi(T, K)}{\partial K^2} \leq 0$$

Interpolating base correlations does not guarantee these conditions hold.

**Worked Example: Arbitrage from naive interpolation**

Suppose the 3% and 7% base tranches have base correlations of 25% and 35% respectively. Linear interpolation gives $\rho^{\text{base}}(5\%) = 30\%$.

Under the Gaussian copula with realistic parameters (5-year maturity, 50bp average spread, 40% recovery):
- $\psi(5Y, 3\%)$ implied by $\rho = 25\%$: approximately 1.2%
- $\psi(5Y, 5\%)$ implied by $\rho = 30\%$: approximately 1.6%
- $\psi(5Y, 7\%)$ implied by $\rho = 35\%$: approximately 2.3%

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

### A6.2.5 The Base Correlation Surface

When 3Y, 5Y, 7Y, and 10Y tranche quotes are available, O'Kane proposes building a **base correlation surface** $\rho(T, K)$:

**Bootstrap approach:**
1. Use 3Y quotes to calibrate $\rho(3, K)$ at standard strikes, assuming piecewise flat structure from $T=0$ to $T=3$
2. For 5Y tranches, solve for flat base correlation between $T=3$ and $T=5$ that reprices the market, using $\rho(3, K)$ for cash flows before $T=3$
3. Repeat for 7Y and 10Y maturities
4. Apply ETL interpolation in strike dimension at each maturity

**No-arbitrage in the time dimension:** The constraint is:
$$\frac{\partial^2 \psi(T, K)}{\partial T \partial K} \geq 0$$

This cross-derivative constraint is difficult to enforce directly. O'Kane suggests enforcing the weaker condition $\partial\psi/\partial T \geq 0$ (ETL increases with maturity), allowing a two-stage construction: strike interpolation first, then maturity interpolation.

### A6.2.6 Crisis Behavior and Regime Dependence

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

**A3. Random factor loading (RFL)**
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

### A6.3.2 Category B: Factor Model Enhancements

**B1. Extended one-factor models**
- Allow time-varying or state-dependent correlation
- Example: correlation increases when factor is low

**B2. Levy factor models**
- Replace Gaussian factor with Levy process
- Generates jumps in default clustering
- More analytically tractable than full dynamic models

**B3. Double-t copula**
- Both factor and idiosyncratic shocks are t-distributed
- Greater flexibility in tail behavior

**B4. Random recovery factor models**
- Recovery rate driven by systematic factor
- $R_i \mid X = f(X)$ where $f$ is decreasing
- Captures "bad states have low recovery" intuition

### A6.3.3 Category C: Dynamic Intensity Models

**C1. Intensity Gamma model** (Joshi and Stacey)
- Time-changed default process with jumps
- Generates default clustering
- Relatively tractable calibration

**C2. Affine Jump Diffusion (AJD)** (Duffie and Garleanu)
- Default intensities follow correlated jump-diffusions
- Common jumps create clustering
- Semi-analytical calibration possible

**C3. Contagion models**
- Defaults increase intensities of surviving credits
- "Infectious default" mechanisms
- Captures crisis propagation

### A6.3.4 Category D: Top-Down and Other Mechanisms

**D1. Markov chain models** (Schönbucher)
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

### A6.3.5 Industry Models: CreditRisk+

While the categories above focus on copula and intensity-based approaches, the industry model **CreditRisk+** (Credit Suisse Financial Products, 1997) takes a fundamentally different approach based on Poisson mixture distributions.

**Structure of CreditRisk+:** McNeil et al. (QRM Section 8.4.2) describe CreditRisk+ as "a Poisson mixture model where the factor vector $\boldsymbol{\Psi}$ consists of $p$ independent, gamma-distributed random variables." The key features are:

1. **Conditional Poisson defaults:** Credit $i$ has a stochastic default intensity $\lambda_i(\boldsymbol{\Psi}) = k_i \boldsymbol{w}_i' \boldsymbol{\Psi}$, where:
   - $k_i \approx$ default probability (for small $k_i$)
   - $\boldsymbol{w}_i$ = vector of factor weights (summing to 1)
   - $\boldsymbol{\Psi}$ = independent gamma-distributed factors with $\text{Ga}(\sigma_j^{-2}, \sigma_j^{-2})$

2. **Gamma-Poisson mixture:** Conditional on $\boldsymbol{\Psi}$, defaults are independent Poisson. The marginal distribution follows from the gamma mixture:

$$\tilde{M} \mid \boldsymbol{\Psi} = \boldsymbol{\psi} \sim \text{Poi}\left(\sum_{i=1}^{m} k_i \boldsymbol{w}_i' \boldsymbol{\psi}\right)$$

McNeil et al. show that for $p$ gamma factors, "$\tilde{M}$ is equal in distribution to a sum of $p$ independent negative binomial random variables."

3. **Panjer recursion:** The loss distribution can be computed efficiently using actuarial techniques. McNeil et al. note: "Using Panjer recursion, it is possible to derive simple recursion formulas for the probabilities $P(\tilde{M} = k)$."

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

## A6.4 Dynamic Bottom-Up Models

### A6.4.1 The Intensity Gamma Model

O'Kane provides extensive treatment of the Intensity Gamma (IG) model of Joshi and Stacey (2005). The key idea is to introduce a time-changed process that generates default clustering.

**The Gamma process driver:** The model uses a gamma process $\Gamma(t)$ to represent "business time"—the arrival of information. A gamma process with shape parameter $\gamma$ and scale parameter $\lambda$ has:
- Increments: $\Gamma(t+s) - \Gamma(t) \sim \text{Gamma}(\gamma s, \lambda)$
- Expected value: $\mathbb{E}[\Gamma(t)] = \gamma t / \lambda$
- Variance: $\text{Var}(\Gamma(t)) = \gamma t / \lambda^2$

**Default mechanism:** Credit $i$ defaults when $\Gamma(t)$ exceeds a random threshold $\theta_i$:
$$\tau_i = \inf\{t: \Gamma(t) \geq \theta_i\}$$

where $\theta_i$ is calibrated to match the marginal survival curve:
$$\mathbb{P}(\theta_i > x) = Q_i(t) \text{ when } \mathbb{E}[\Gamma(t)] = x$$

**Why this generates clustering:** Large jumps in $\Gamma(t)$ can cause multiple thresholds to be exceeded simultaneously, generating clustered defaults. O'Kane demonstrates this with simulation: "The intensity gamma model exhibits default clustering... we see that defaults do not occur in isolation but tend to occur in groups."

**Calibration:** The model has $2n$ parameters if $n$ gamma processes are used. For a single gamma process:
$$\Gamma(t) = \Gamma_1(t) + \Gamma_2(t)$$

O'Kane reports calibration to CDX NA IG tranches with parameters: $\gamma_1 = 0.0008$, $\gamma_2 = 0.217$, $\lambda_1 = 0.0011$, $\lambda_2 = 0.186$.

### A6.4.2 The Affine Jump Diffusion Model

The AJD model of Duffie and Garleanu (1999) models default intensity dynamics directly:

$$dX(t) = \kappa(\theta - X(t))dt + \sigma\sqrt{X(t)}dW(t) + J \, dN(t)$$

where:
- $\kappa(\theta - X)$ is mean-reversion
- $\sigma\sqrt{X}dW$ is diffusive volatility (CIR-type)
- $J \, dN$ is a jump process with intensity $\ell$ and jump size distribution

**Credit-specific intensities:** Each credit $i$ has intensity:
$$\lambda_i(t) = \alpha_i + \beta_i X(t)$$

where $\alpha_i$ is idiosyncratic and $\beta_i X(t)$ is systematic exposure.

**Common vs. idiosyncratic jumps:** The jump process can be decomposed:
$$J \, dN = J_c \, dN_c + J_i \, dN_i$$

where $N_c$ is a common jump process affecting all credits.

**Analytical tractability:** O'Kane shows that survival probabilities can be computed analytically:
$$\mathbb{E}\left[\exp\left(-\int_0^T X(s)ds\right)\right] = \exp(A(T) + B(T)X(0))$$

where $A(T)$ and $B(T)$ satisfy Riccati ODEs.

**Calibration results:** Mortensen (2006), cited by O'Kane, reports calibration to CDX NA IG with parameters: $\kappa = 0.20$, $\sigma = 0.054$, $\ell = 0.037$, $\mu = 0.067$, $w = 0.93$ (common jump fraction).

### A6.4.3 Contagion Models

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

---

## A6.5 Top-Down Dynamic Models

### A6.5.1 The Markov Chain Approach

Schönbucher's Markov chain model (2005) directly models the portfolio loss process $L(t)$ as a continuous-time Markov chain on states $\{0, 1, 2, \ldots, N_C\}$.

**Generator matrix:** The dynamics are governed by the generator matrix $A(t)$ with elements $a_{ij}(t)$ representing the rate of transition from $i$ to $j$ defaults.

For a single-step model (one default at a time):
$$A(t) = \begin{pmatrix}
-a_0(t) & a_0(t) & 0 & \cdots \\
0 & -a_1(t) & a_1(t) & \cdots \\
\vdots & & \ddots & \\
\end{pmatrix}$$

**No-arbitrage conditions:** The transition rates must satisfy:
- $a_{ij}(t) \geq 0$ for $i < j$
- $a_{ij}(t) = 0$ for $i > j$ (no "un-defaults")
- Consistency with observed tranche prices

**Calibration:** O'Kane demonstrates calibration to CDX NA IG tranches by solving for the vector $(a_0, a_1, \ldots, a_{124})$. With 125 parameters and 5 tranches plus the index, exact calibration is possible with many degrees of freedom.

**Stochastic generator extension:** For pricing forward-starting products, the generator itself can be made stochastic:
$$da_n(t) = \mu_n(t)dt + \sigma_n(t)dW_n(t)$$

This allows modeling of spread dynamics and MTM risk.

### A6.5.2 Loss Process Intensity Models

An alternative top-down approach models the intensity of the loss process itself:

$$\lambda^L(t) = f(L(t), X(t))$$

where the loss intensity depends on current losses and a systematic factor.

**Self-exciting feature:** If $\partial f / \partial L > 0$, the model exhibits contagion—defaults increase the rate of future defaults.

---

## A6.6 Vasicek Model and Regulatory Applications

### A6.6.1 The Vasicek WCDR Formula

Hull in *Risk Management and Financial Institutions* develops the Vasicek model for credit portfolio risk, which underlies the Basel II/III internal ratings-based (IRB) approach.

For a large homogeneous portfolio with one-year default probability $p$ and asset correlation $\rho$, the **Worst Case Default Rate** at confidence level $X$ is:

$$\boxed{\text{WCDR}(T, X) = \Phi\left(\frac{\Phi^{-1}(p) + \sqrt{\rho}\Phi^{-1}(X)}{\sqrt{1-\rho}}\right)}$$

Hull explains: "This is a strange-looking result, but a very important one. It was first developed by Vasicek in 1987."

**Intuition:** At the $X$-th percentile of the systematic factor distribution, the conditional default probability is WCDR. This is the default rate that will not be exceeded with probability $X$.

**Worked Example (Hull Example 11.2):**
- Portfolio: Large number of retail loans
- One-year PD: $p = 2\%$
- Asset correlation: $\rho = 0.10$
- Confidence level: $X = 99.9\%$

$$\text{WCDR}(1, 0.999) = \Phi\left(\frac{\Phi^{-1}(0.02) + \sqrt{0.10} \times \Phi^{-1}(0.999)}{\sqrt{1-0.10}}\right)$$
$$= \Phi\left(\frac{-2.054 + 0.316 \times 3.090}{0.949}\right)$$
$$= \Phi(0.863) = 0.128$$

The 99.9% worst-case one-year default rate is **12.8%**, dramatically higher than the 2% expected default rate.

### A6.6.2 Basel II/III Capital Formula

The IRB capital requirement for credit risk uses a modified Vasicek formula:

$$K = \text{LGD} \times \left[\text{WCDR}(1, 0.999) - p\right] \times \text{MA}$$

where:
- LGD is loss-given-default
- MA is a maturity adjustment factor
- The subtraction of $p$ accounts for expected loss (covered by provisions)

**Correlation function:** Basel specifies correlation as a function of PD:

$$\rho = 0.12 \times \frac{1 - e^{-50 \times p}}{1 - e^{-50}} + 0.24 \times \left(1 - \frac{1 - e^{-50 \times p}}{1 - e^{-50}}\right)$$

This gives $\rho \approx 0.24$ for very low PDs and $\rho \approx 0.12$ for high PDs—recognizing that high-PD obligors have more idiosyncratic risk.

---

## A6.7 Calibration and Validation

### A6.7.1 Calibration Targets

Credit portfolio models can be calibrated to:

| Target | Information Content | Availability |
|--------|---------------------|--------------|
| Single-name CDS | Marginal default probabilities | Liquid, daily |
| Index tranches | Joint loss distribution at traded strikes | Liquid, daily |
| Bespoke tranches | Information at specific strikes | Illiquid |
| Historical defaults | Real-world default clustering | Long history needed |
| Equity correlations | Proxy for asset correlations | Requires model assumptions |

### A6.7.2 Calibration Approaches

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

### A6.7.3 Model Comparison and Selection

McNeil et al. emphasize "model risk issues" in credit portfolio modeling. Key considerations:

**In-sample fit:** Does the model price traded tranches accurately?

**Out-of-sample prediction:** Does the model price non-traded tranches consistently?

**Parameter stability:** Do calibrated parameters remain stable through time?

**Economic reasonableness:** Are implied correlations and intensities plausible?

**Hedge performance:** Do delta hedges computed from the model work in practice?

### A6.7.4 Validation Tests

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

## A6.8 Numerical Implementation

### A6.8.1 Loss Distribution Computation

**Exact recursion (O'Kane Chapter 18):**

For heterogeneous portfolios, the conditional loss distribution is computed iteratively:

1. Start with $f^{(0)}(k) = \mathbf{1}_{\{k=0\}}$ (no credits added)
2. For each credit $j = 1, \ldots, N_C$:
   $$f^{(j)}(k) = f^{(j-1)}(k)(1 - p_j(T \mid X)) + f^{(j-1)}(k-1)p_j(T \mid X)$$
3. Integrate over the factor: $f(k) = \int f^{(N_C)}(k \mid X)\phi(X)dX$

**Computational complexity:** $O(N_C^2 \times N_X)$ where $N_X$ is the number of factor quadrature points.

**Efficiency improvement:** Only compute up to the strike of interest: $\min(g, j)$ where $g = \lceil K_2 / u \rceil$.

### A6.8.2 Approximations for Large Portfolios

**Large Homogeneous Portfolio (LHP):**
$$F_L(\ell) = \Phi\left(\frac{\sqrt{1-\rho}\Phi^{-1}(\ell/(1-R)) - \Phi^{-1}(p)}{\sqrt{\rho}}\right)$$

**Gaussian approximation (Shelton 2004):**
- Approximate conditional loss distribution by Gaussian with matched mean and variance
- Closed-form tranche expected loss

**Adjusted binomial (Andersen, Sidenius, Basu 2003):**
- Match first three moments to a shifted binomial
- More accurate than pure Gaussian for finite portfolios

O'Kane compares these methods for CDX and CDX HY portfolios, finding that the exact recursion is necessary for accuracy at equity strikes but approximations suffice for senior tranches.

### A6.8.3 Monte Carlo Methods

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

---

## A6.9 Practical Notes

### A6.9.1 Model Risk in Credit Portfolios

**The model risk is large.** O'Kane emphasizes that correlation products "have some risk which we are not able to hedge using the range of available market securities." This unhedgeable model risk requires:
- Conservative reserves
- Stress testing across models
- Transparency about model limitations

**The 2008 lesson:** Base correlation's failures were foreseeable from its theoretical limitations. Model users must understand not just how to use a model, but *when it will fail*.

### A6.9.2 Regulatory Implications

**IRB capital:** Banks using internal models for regulatory capital face constraints:
- Correlation functions are prescribed (limited flexibility)
- Model validation requirements
- Conservative parameter choices

**Stress testing:** Post-2008 regulations require stress testing of correlation assumptions. Models should be evaluated at:
- Historical stress correlations
- Expert-specified adverse scenarios
- Reverse stress tests ("what breaks the book?")

### A6.9.3 Implementation Pitfalls

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

## A6.10 Worked Examples

### Example A6.1: Base Correlation Calibration

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
   - Protection leg PV depends on $\psi(5Y, 3\%)$ under correlation $\rho$
   - Premium leg PV depends on tranche survival curve
   - Root-finding gives $\rho^{\text{base}}(3\%) \approx 23\%$

2. **3-7% tranche:** Must first compute $\rho^{\text{base}}(7\%)$
   - The 3-7% tranche is: $[0-7\%] - [0-3\%]$
   - Solve for $\rho^{\text{base}}(7\%)$ such that the net position prices correctly
   - Result: $\rho^{\text{base}}(7\%) \approx 38\%$

3. Continue for higher strikes...

**Final base correlation curve:**

| Strike | Base Correlation |
|--------|------------------|
| 3% | 23% |
| 7% | 38% |
| 10% | 48% |
| 15% | 57% |
| 30% | 74% |

### Example A6.2: Vasicek WCDR Calculation

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
$$\text{VaR}_{99.9\%} = \text{WCDR} \times \text{LGD} \times \text{Portfolio Notional}$$

If portfolio notional is \$100 million:
$$\text{VaR}_{99.9\%} = 0.145 \times 0.45 \times \$100M = \$6.525M$$

3. **Expected loss for comparison:**
$$\text{EL} = p \times \text{LGD} \times \text{Notional} = 0.015 \times 0.45 \times \$100M = \$0.675M$$

The unexpected loss (VaR minus EL) is $\$6.525M - \$0.675M = \$5.85M$.

### Example A6.3: Intensity Gamma Model Simulation

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
- Since $\Gamma(5) = 1.2 > 0.6$, credit 1 defaults
- Repeat for all credits

The simulation exhibits **clustering**: when $\Gamma(5)$ is large, many credits default; when small, few default.

### Example A6.4: Tranche Delta in the Gaussian Copula

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

2. **Static extensions:** Alternative copulas (t, Clayton), random factor loading, and stochastic recovery can address some limitations while preserving tractability.

3. **Dynamic intensity models:** The Intensity Gamma and Affine Jump Diffusion models provide explicit dynamics for default clustering and spread evolution.

4. **Top-down models:** Markov chain approaches model the portfolio loss process directly, enabling natural calibration to tranche prices and pricing of forward-starting products.

5. **Regulatory models:** The Vasicek WCDR formula underlies Basel IRB capital requirements, providing a link between portfolio credit models and bank regulation.

6. **Model risk:** All credit portfolio models have significant limitations. Users must understand not just how to use models, but when they will fail.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Portfolio loss distribution** | Probability law of aggregate losses | Drives tranche pricing and risk |
| **Tail dependence** | Probability of joint extreme events | Senior tranche risk; Gaussian copula has none |
| **Base correlation** | Correlation implied by equity tranches | Market quoting convention; not consistent model |
| **Conditional independence** | Independence given systematic factor | Enables tractable computation |
| **WCDR** | Worst Case Default Rate at confidence level | Regulatory capital formula |
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
| 26 | What is the correlation formula in a two-factor model? | $c_{ij} = \beta_{1i}\beta_{1j} + \beta_{2i}\beta_{2j}$ |
| 27 | How many factors are needed to model $M$ sectors? | $M$ factors—one-factor models cannot capture sector structure |
| 28 | What happens to credit B's spread when positively correlated credit A defaults? | B's spread jumps up; the magnitude increases with correlation |
| 29 | What is ETL interpolation? | Interpolate expected tranche loss directly, then invert to base correlation |
| 30 | Why is ETL interpolation preferred over base correlation interpolation? | No-arbitrage constraints ($\partial\psi/\partial K \in [0,1]$, convexity) are easier to enforce in ETL space |
| 31 | What is the PCHIP spline? | Piecewise Cubic Hermite Interpolant: guarantees smoothness and monotonicity |
| 32 | What is the time-dimension no-arbitrage constraint for base correlation surface? | $\partial^2\psi/\partial T\partial K \geq 0$ |
| 33 | What is the Clayton copula's lower tail dependence coefficient? | $\lambda_L = 2^{-1/\theta}$ for $\theta > 0$; e.g., $\theta = 2$ gives $\lambda_L \approx 0.71$ |
| 34 | What is CreditRisk+? | A Poisson mixture model with gamma-distributed factors; defaults are conditionally Poisson |
| 35 | What distribution does the number of defaults follow in CreditRisk+? | Sum of independent negative binomial random variables (one per gamma factor) |
| 36 | How does CreditRisk+ connect to copula models? | With gamma factors, it's equivalent to a generalized LT-Archimedean factor copula model |

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

### Problem 3: Base Correlation Arbitrage
Given base correlations: $\rho^B(3\%) = 25\%$, $\rho^B(7\%) = 40\%$. Linear interpolation gives $\rho^B(5\%) = 32.5\%$.
(a) Compute the expected tranche losses $\psi(5Y, K)$ for $K = 3\%, 5\%, 7\%$
(b) Check whether the convexity condition holds
(c) Describe an arbitrage if it doesn't

### Problem 4: Intensity Gamma Calibration
The IG model with single gamma process has parameters $(\gamma, \lambda)$. Given portfolio average spread 50bp and 5-year base correlation skew:
- $\rho^B(3\%) = 25\%$
- $\rho^B(7\%) = 38\%$

(a) What is the calibration target (what equation must be satisfied)?
(b) Why might a single gamma process be insufficient?
(c) What happens if we add a second gamma process?

### Problem 5: Markov Chain Generator
For a 100-credit portfolio, the Markov chain model has transition rates $(a_0, a_1, \ldots, a_{99})$.
(a) If $a_n = 0.1$ for all $n$, what is the expected number of defaults by $T = 5$?
(b) If $a_n = 0.1 \times (1 + 0.01n)$, how does this change the loss distribution?
(c) Interpret the economic meaning of increasing transition rates.

### Problem 6: Multi-Factor Model
Extend the one-factor model to two factors (industry $X_1$ and market $X_2$):
$$Z_i = \beta_{i1}X_1 + \beta_{i2}X_2 + \sqrt{1 - \beta_{i1}^2 - \beta_{i2}^2}\varepsilon_i$$

(a) What is the correlation between credits in the same industry? Different industries?
(b) How many parameters are needed for 125 credits in 10 industries?
(c) What information could calibrate the industry-specific loadings?

### Problem 7: Dynamic Model Spread Dynamics
In the AJD model, suppose the common intensity factor $X(t)$ jumps by $\Delta X = 0.5$ due to a credit event.
(a) How does the survival probability of remaining credits change?
(b) How does the expected tranche loss change?
(c) Why does this matter for mark-to-market risk?

### Problem 8: Loss Distribution Computation
For a 10-credit portfolio with:
- Identical marginals: $Q_i(5) = 0.90$
- Flat correlation: $\rho = 0.30$
- Equal weights: $w_i = 0.10$
- Zero recovery: $R = 0$

(a) Use the recursion to compute $\mathbb{P}(L(5) = 0)$, $\mathbb{P}(L(5) = 0.1)$, $\mathbb{P}(L(5) = 0.2)$
(b) Compare to the independent case ($\rho = 0$)
(c) Compute the 0-10% equity tranche expected loss

### Problem 9: ETL Interpolation
Given base correlations at standard strikes:
- $\rho^B(3\%) = 22\%$ → $\psi(5Y, 3\%) = 1.1\%$
- $\rho^B(7\%) = 35\%$ → $\psi(5Y, 7\%) = 2.0\%$
- $\rho^B(10\%) = 45\%$ → $\psi(5Y, 10\%) = 2.5\%$

(a) Use linear interpolation in ETL space to compute $\psi(5Y, 5\%)$
(b) Check the convexity condition: is $\partial^2\psi/\partial K^2 \leq 0$?
(c) Compare to linear interpolation in base correlation space

### Problem 10: Default-Induced Spread Dynamics
Credits A and B have flat 5Y survival probability $Q_A(5) = Q_B(5) = 0.95$ and asset correlation $\rho = 0.40$.

(a) Compute the default thresholds $C_A(5)$ and $C_B(5)$
(b) If A defaults at $t = 0$ (instantaneous), what is B's conditional survival probability to $T = 5$?
(c) How much does B's 5Y spread change after A's default?

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Base correlation is "not a pricing model but rather a quoting convention" | O'Kane Ch 20 |
| Gaussian copula has zero tail dependence for $\rho < 1$ | McNeil et al. QRM Ch 5 |
| t-copula tail dependence formula | McNeil et al. QRM Example 5.33 |
| WCDR formula | Hull RMFI Ch 11; Vasicek (1987) |
| Intensity Gamma model and calibration | O'Kane Ch 23; Joshi and Stacey (2005) |
| AJD model structure | O'Kane Ch 23; Duffie and Garleanu (1999) |
| Markov chain approach | O'Kane Ch 24; Schönbucher (2005) |
| No-arbitrage conditions on ETL | O'Kane Ch 19 |
| Loss distribution recursion | O'Kane Ch 18 |
| Basel II correlation function | Hull RMFI Ch 15; Basel Committee |
| Conditional independence in factor models | O'Kane Ch 13; McNeil et al. QRM Ch 8 |
| CreditRisk+ Poisson mixture model | McNeil et al. QRM Section 8.4.2 |
| CreditRisk+ as sum of negative binomials | McNeil et al. QRM Section 8.4.2 |
| CreditRisk+ connection to LT-Archimedean copula | McNeil et al. QRM |
| Panjer recursion for CreditRisk+ | McNeil et al. QRM Section 8.4.2 |
| Clayton copula lower tail dependence $\lambda_L = 2^{-1/\theta}$ | McNeil et al. QRM Example 5.31 |
| Gumbel copula upper tail dependence $\lambda_U = 2 - 2^{1/\theta}$ | McNeil et al. QRM Example 5.31 |
| Multi-factor correlation formula $c_{ij} = \beta_{1i}\beta_{1j} + \beta_{2i}\beta_{2j}$ | O'Kane Ch 13 Section 13.5 |
| Sector correlation matrix example (52%/16%/61%) | O'Kane Ch 13 |
| "M sectors requires M-factor model" | O'Kane Ch 13 |
| Default-induced spread dynamics formula | O'Kane Ch 13 Section 13.6 |
| Conditional survival curve after default | O'Kane Ch 13 Equation 13.3 |
| Spread jump observations (positive/negative correlation) | O'Kane Ch 13 Figure 13.7 |
| ETL interpolation methodology | O'Kane Ch 20 Section 20.5 |
| PCHIP spline for monotonic interpolation | O'Kane Ch 20; Fritsch and Carlson (1980) |
| Base correlation surface bootstrap | O'Kane Ch 20 Section 20.6 |
| Time dimension no-arbitrage constraint $\partial^2\psi/\partial T\partial K \geq 0$ | O'Kane Ch 20 |
| "ETL interpolation is more arbitrage free than direct interpolation" | O'Kane Ch 20 |

### (B) Reasoned Inference (Derived from A)

- The correlation smile's interpretation as "missing tail dependence" follows from combining O'Kane's observations about base correlation failures with McNeil's tail dependence analysis
- The arbitrage from naive interpolation follows from applying O'Kane's no-arbitrage conditions to linear interpolation
- Regulatory capital implications follow from combining Hull's WCDR treatment with Basel framework structure

### (C) Flagged Uncertainties

- **Dynamic model adoption:** O'Kane notes "it is still too early to say which, if any of these models, will become widely adopted." I'm not sure about current market standard practice for dynamic models
- **Contagion calibration:** While contagion mechanisms are theoretically motivated, I'm not sure how practitioners typically calibrate infection probabilities in Davis-Lo type models
- **Post-crisis evolution:** The treatment reflects pre-2008 and immediate post-crisis literature. Market practice may have evolved further—recommend verifying current conventions

---

*Last Updated: January 2026*
